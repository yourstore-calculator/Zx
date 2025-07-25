# Zx
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <title>حاسبة الأسعار</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background: #f4f4f4;
      color: #333;
      direction: rtl;
      padding: 20px;
    }
    h1, h2 {
      text-align: center;
    }
    .logo {
      text-align: center;
      margin-bottom: 20px;
    }
    .logo img {
      max-width: 160px;
    }
    label, select, input {
      display: block;
      margin: 8px 0;
      width: 100%;
      padding: 5px;
    }
    .section {
      background: #fff;
      padding: 15px;
      margin-bottom: 20px;
      border-radius: 8px;
      box-shadow: 0 0 5px #ccc;
    }
    .results {
      background: #fff;
      padding: 15px;
      border-radius: 8px;
      box-shadow: 0 0 5px #ccc;
    }
    .results div {
      margin: 6px 0;
    }
    .totals {
      margin-top: 20px;
      padding: 10px;
      background: #eee;
      font-weight: bold;
    }
    button {
      padding: 10px 15px;
      background: black;
      color: white;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      margin-top: 10px;
    }
    button:hover {
      background: #333;
    }
    .flex {
      display: flex;
      gap: 10px;
    }
    .half {
      flex: 1;
    }
  </style>
</head>
<body>

<div class="logo">
  <img src="https://i.imgur.com/ih5zjVj.png" alt="الشعار">
</div>

<h1>🔧 حاسبة الأسعار</h1>

<div class="section">
  <label for="category">اختر القسم:</label>
  <select id="category">
    <option>اختر</option>
    <option>نوافذ</option>
    <option>أبواب</option>
    <option>أبواب السحب</option>
  </select>

  <label for="subcategory">اختر النوع:</label>
  <select id="subcategory"></select>

  <label for="type">اختر النوع الفرعي:</label>
  <select id="type"></select>

  <div class="flex">
    <div class="half">
      <label for="height">الطول (متر):</label>
      <input type="number" id="height" placeholder="مثال: 2.2" step="0.01">
    </div>
    <div class="half">
      <label for="width">العرض (متر):</label>
      <input type="number" id="width" placeholder="مثال: 1.2" step="0.01">
    </div>
  </div>

  <label for="quantity">الكمية:</label>
  <input type="number" id="quantity" value="1" min="1">

  <div id="addons">
    <label><input type="checkbox" id="curtain"> إضافة ستارة داخلية</label>
    <div class="flex" id="curtainSize" style="display:none;">
      <div class="half">
        <input type="number" id="curtainHeight" placeholder="طول الستارة" step="0.01">
      </div>
      <div class="half">
        <input type="number" id="curtainWidth" placeholder="عرض الستارة" step="0.01">
      </div>
    </div>

    <label><input type="checkbox" id="mesh"> إضافة شبك</label>
    <select id="meshType" style="display:none;">
      <option>ثابت</option>
      <option>سلايد</option>
      <option>فولدينج</option>
      <option>باب</option>
    </select>
  </div>

  <button onclick="calculate()">🔍 احسب</button>
  <button onclick="clearResults()">🗑️ مسح النتائج</button>
  <button onclick="saveAsWord()">💾 حفظ كـ Word</button>
</div>

<div class="results" id="results">
  <h2>📋 النتائج:</h2>
  <div id="resultsList"></div>
  <div class="totals" id="summary"></div>
</div>

<script>
  // الكود البرمجي الخاص بالحساب والإضافات
  // سيتم إرساله في الرد القادم مباشرة (لأنconst prices = {
  "نوافذ": {
    "دبل جلاس دبل فريم": 73,
    "دبل جلاس سنجل فريم": 46,
    "سنجل جلاس سنجل فريم": 43,
    "نوافذ السلايدنج": 10, // سعر إضافي
    "النوافذ الكهربائية": 102,
    "سكاي لايت بدون مكينة": 56,
    "سكاي لايت مع مكينة": 145,
    "كارتن وول ثقيل": 56,
    "كارتن وول خفيف": 45,
  },
  "أبواب": {
    "باب مدخل زينك": 66,
    "باب مدخل ستينلس ستيل": 120,
    "باب مدخل كاست المنيوم": 168,
    "WPC فارغ": 45,
    "WPC مع خشب": 50,
    "WPC مع حشوة ضد الصوت": 60,
    "WPC مع فريم ألمنيوم": 67,
    "WPC سلايدنج": 65,
    "ألمنيوم فارغ": 65,
    "ألمنيوم مع خشب": 75,
    "ألمنيوم فل ألمنيوم": 85,
    "ألمنيوم مخفي": 110,
    "ألمنيوم خارجي": 61,
    "دورات المياه النوع الجديد": 55,
    "دورات المياه النوع الأقدم": 45,
    "دورات المياه مخفي زجاجي": 65
  },
  "أبواب السحب": {
    "داخلي زجاج": 38,
    "داخلي متين": 41,
    "خارجي جزء مفتوح": 55,
    "خارجي جزئين مفتوحات": 58,
    "WPC سلايد": 61,
    "فولدنج داخلي": 39,
    "فولدنج خارجي": 56,
  }
};

const factors = {
  "نوافذ": 0.13,
  "أبواب": {
    "باب مدخل": 0.2,
    "باب حديقة": 0.2,
    "افتراضي": 0.11
  },
  "أبواب السحب": {
    "WPC": 0.11,
    "افتراضي": 0.13
  }
};

const curtainPrice = 26; // ريال للمتر المربع
const commissionPercent = 0.04;

document.getElementById("category").addEventListener("change", function () {
  const cat = this.value;
  const sub = document.getElementById("subcategory");
  sub.innerHTML = "";

  if (cat === "نوافذ") {
    sub.innerHTML += `<option>دبل جلاس دبل فريم</option>
                      <option>دبل جلاس سنجل فريم</option>
                      <option>سنجل جلاس سنجل فريم</option>
                      <option>نوافذ السلايدنج</option>
                      <option>النوافذ الكهربائية</option>
                      <option>سكاي لايت بدون مكينة</option>
                      <option>سكاي لايت مع مكينة</option>
                      <option>كارتن وول ثقيل</option>
                      <option>كارتن وول خفيف</option>`;
  } else if (cat === "أبواب") {
    sub.innerHTML += `<option>باب مدخل زينك</option>
                      <option>باب مدخل ستينلس ستيل</option>
                      <option>باب مدخل كاست المنيوم</option>
                      <option>WPC فارغ</option>
                      <option>WPC مع خشب</option>
                      <option>WPC مع حشوة ضد الصوت</option>
                      <option>WPC مع فريم ألمنيوم</option>
                      <option>WPC سلايدنج</option>
                      <option>ألمنيوم فارغ</option>
                      <option>ألمنيوم مع خشب</option>
                      <option>ألمنيوم فل ألمنيوم</option>
                      <option>ألمنيوم مخفي</option>
                      <option>ألمنيوم خارجي</option>
                      <option>دورات المياه النوع الجديد</option>
                      <option>دورات المياه النوع الأقدم</option>
                      <option>دورات المياه مخفي زجاجي</option>`;
  } else if (cat === "أبواب السحب") {
    sub.innerHTML += `<option>داخلي زجاج</option>
                      <option>داخلي متين</option>
                      <option>خارجي جزء مفتوح</option>
                      <option>خارجي جزئين مفتوحات</option>
                      <option>WPC سلايد</option>
                      <option>فولدنج داخلي</option>
                      <option>فولدنج خارجي</option>`;
  }
});

document.getElementById("curtain").addEventListener("change", function () {
  document.getElementById("curtainSize").style.display = this.checked ? "flex" : "none";
});
document.getElementById("mesh").addEventListener("change", function () {
  document.getElementById("meshType").style.display = this.checked ? "block" : "none";
});

function calculate() {
  const cat = document.getElementById("category").value;
  const sub = document.getElementById("subcategory").value;
  const height = parseFloat(document.getElementById("height").value || "0");
  const width = parseFloat(document.getElementById("width").value || "0");
  const qty = parseInt(document.getElementById("quantity").value || "1");

  const basePrice = prices[cat]?.[sub] || 0;
  const area = (height * width).toFixed(2);

  // تسعير الستارة
  let curtainCost = 0;
  if (document.getElementById("curtain").checked) {
    const ch = parseFloat(document.getElementById("curtainHeight").value || "0");
    const cw = parseFloat(document.getElementById("curtainWidth").value || "0");
    curtainCost = (ch * cw * curtainPrice);
  }

  // تسعير الشبك - مجرد ظهور فقط
  const meshText = document.getElementById("mesh").checked ? ` + شبك (${document.getElementById("meshType").value})` : "";

  // تحديد عامل الشحن
  let factor = 0.13;
  if (cat === "أبواب") {
    if (sub.includes("مدخل") || sub.includes("حديقة")) factor = 0.2;
    else factor = 0.11;
  } else if (cat === "أبواب السحب") {
    factor = sub.includes("WPC") ? 0.11 : 0.13;
  }

  // تثبيت المقاس لأبواب WPC - الألمنيوم - دورات المياه
  const fixedTypes = ["WPC", "ألمنيوم", "دورات المياه"];
  if (fixedTypes.some(type => sub.includes(type))) {
    if (height !== 2.2 || width !== 1) {
      alert("المقاس يجب أن يكون 2.2 × 1 لهذه الأنواع");
      return;
    }
  }

  const shipping = (area * factor * 48);
  const pricePerItem = (basePrice * area) + shipping + curtainCost;
  const total = pricePerItem * qty;
  const commission = total * commissionPercent;
  const final = total + commission;

  const resultDiv = document.getElementById("resultsList");
  const res = document.createElement("div");
  res.innerHTML = `🚪 النوع: <b>${sub}</b><br>📐 المقاس: ${height} × ${width}<br>🔢 الكمية: ${qty}<br>🚚 الشحن: ${shipping.toFixed(2)} ريال<br>💵 الإجمالي: ${(total).toFixed(2)} ريال ${meshText}`;
  resultDiv.appendChild(res);

  updateTotals();
}

function updateTotals() {
  const results = document.querySelectorAll("#resultsList div");
  let total = 0;

  results.forEach(div => {
    const match = div.innerHTML.match(/الإجمالي: ([\d.]+)/);
    if (match) total += parseFloat(match[1]);
  });

  const commission = total * commissionPercent;
  const final = total + commission;

  document.getElementById("summary").innerHTML =
    `🚚💵 <br>المجموع: ${total.toFixed(2)} ريال<br>العمولة (${commissionPercent * 100}%): ${commission.toFixed(2)} ريال<br>الناتج النهائي: ${final.toFixed(2)} ريال<br><br><small>*الأسعار شاملة الشحن<br>**الأسعار لا تشمل التركيب</small>`;
}

function clearResults() {
  document.getElementById("resultsList").innerHTML = "";
  document.getElementById("summary").innerHTML = "";
}

function saveAsWord() {
  const content = document.getElementById("results").innerHTML;
  const header = "<html><head><meta charset='utf-8'></head><body dir='rtl'>" + content + "</body></html>";
  const blob = new Blob([header], { type: "application/msword" });
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = "النتائج.doc";
  link.click();
} الكود طويل)
</script>

</body>
</html>
