<!DOCTYPE html>
<html lang="th">

<head>

  <meta charset="UTF-8">

  <meta
    name="viewport"
    content="width=device-width, initial-scale=1.0"
  >

  <title>เชื่อมบัญชีร้านค้า | BlueTap</title>

  <script src="https://static.line-scdn.net/liff/edge/2/sdk.js"></script>


  <style>

    * {
      box-sizing: border-box;
    }


    body {

      margin: 0;

      font-family:
        -apple-system,
        BlinkMacSystemFont,
        "Segoe UI",
        Arial,
        sans-serif;

      background: #f5f7fa;

      color: #172033;

    }


    .container {

      min-height: 100vh;

      display: flex;

      justify-content: center;

      align-items: center;

      padding: 24px;

    }


    .card {

      width: 100%;

      max-width: 420px;

      background: #ffffff;

      border-radius: 20px;

      padding: 32px 28px;

      box-shadow:
        0 8px 30px rgba(0, 0, 0, 0.08);

    }


    .logo {

      text-align: center;

      font-size: 24px;

      font-weight: 700;

      margin-bottom: 28px;

    }


    .title {

      font-size: 22px;

      font-weight: 700;

      margin-bottom: 8px;

    }


    .description {

      font-size: 14px;

      line-height: 1.6;

      color: #687386;

      margin-bottom: 28px;

    }


    .label {

      display: block;

      font-size: 14px;

      font-weight: 600;

      margin-bottom: 8px;

    }


    input {

      width: 100%;

      padding: 14px 16px;

      border: 1px solid #d9dee7;

      border-radius: 10px;

      font-size: 16px;

      outline: none;

    }


    input:focus {

      border-color: #2c6bed;

      box-shadow:
        0 0 0 3px rgba(44, 107, 237, 0.1);

    }


    .hint {

      margin-top: 8px;

      font-size: 12px;

      color: #8a93a3;

    }


    button {

      width: 100%;

      margin-top: 24px;

      padding: 14px;

      border: none;

      border-radius: 10px;

      background: #1f5eff;

      color: white;

      font-size: 16px;

      font-weight: 600;

      cursor: pointer;

    }


    button:disabled {

      opacity: 0.6;

      cursor: not-allowed;

    }


    #result {

      text-align: center;

      margin-top: 18px;

      font-size: 14px;

      line-height: 1.5;

    }


    .success {

      color: #16803c;

    }


    .error {

      color: #d93025;

    }


    .loading {

      color: #687386;

    }


    .footer {

      text-align: center;

      margin-top: 24px;

      font-size: 11px;

      color: #9aa3b2;

      line-height: 1.5;

    }

  </style>

</head>


<body>


<div class="container">


  <div class="card">


    <div class="logo">
      BlueTap
    </div>


    <div class="title">
      เชื่อมบัญชีร้านค้า
    </div>


    <div class="description">

      กรุณากรอก Merchant ID
      เพื่อเชื่อมบัญชีร้านค้าของคุณกับ LINE OA

    </div>


    <label
      class="label"
      for="merchantId"
    >
      Merchant ID
    </label>


    <input
      type="text"
      id="merchantId"
      placeholder="กรอก Merchant ID"
      inputmode="text"
      maxlength="10"
      autocomplete="off"
    >


    <div class="hint">

      กรุณากรอก Merchant ID ให้ครบ 10 ตัวอักษร

    </div>


    <button
      id="submitButton"
      onclick="submitForm()"
    >

      ยืนยันข้อมูล

    </button>


    <div id="result"></div>


    <div class="footer">

      ข้อมูลร้านค้าและข้อมูล LINE
      จะถูกใช้สำหรับการเชื่อมบัญชีเท่านั้น

    </div>


  </div>

</div>


<script>


// ==========================================
// LIFF ID
// ==========================================

const LIFF_ID =
  "2011137760-Da2862OU";


// ==========================================
// GOOGLE APPS SCRIPT  APP
// ==========================================
//
// IMPORTANT:
// ต้องใช้ URL ที่ลงท้ายด้วย /exec
//
// ตัวอย่าง:
// https://script.google.com/macros/s/XXXXX/exec
//
// อย่าใช้ /dev สำหรับการใช้งานจริง
//

const WEB_APP_URL =
  "https://script.google.com/macros/s/AKfycbwiXZYczfQUlyl9gM98ozoziseAtmylOxIN-VGDx9bBLl4w0kMvQp82dFrqaDS6Jtt4/exec";


// ==========================================
// INITIALIZE LIFF
// ==========================================

async function main() {

  try {

    await liff.init({

      liffId: LIFF_ID

    });


    if (!liff.isLoggedIn()) {

      liff.login();

      return;

    }


  } catch (error) {

    console.error(error);

    showResult(

      "ไม่สามารถเชื่อมต่อกับ LINE ได้ กรุณาลองใหม่",

      "error"

    );

  }

}


// ==========================================
// SUBMIT FORM
// ==========================================

async function submitForm() {


  const merchantId =

    document

      .getElementById("merchantId")

      .value

      .trim();


  const button =

    document.getElementById(

      "submitButton"

    );


  // ========================================
  // CHECK MERCHANT ID
  // ========================================

  if (

    merchantId.length !== 10

  ) {

    showResult(

      "กรุณากรอก Merchant ID ให้ครบ 10 ตัวอักษร",

      "error"

    );

    return;

  }


  try {


    button.disabled = true;


    button.innerText =

      "กำลังบันทึก...";


    showResult(

      "กำลังบันทึกข้อมูล...",

      "loading"

    );


    // ======================================
    // GET LINE PROFILE
    // ======================================

    const profile =

      await liff.getProfile();


    const userId =

      profile.userId;


    const displayName =

      profile.displayName;


    // ======================================
    // BUILD REQUEST URL
    // ======================================

    const requestUrl =

      WEB_APP_URL +

      "?userId=" +

      encodeURIComponent(userId) +

      "&displayName=" +

      encodeURIComponent(displayName) +

      "&merchantId=" +

      encodeURIComponent(merchantId);


    console.log(

      "Sending data to:",

      requestUrl

    );


    // ======================================
    // SEND DATA TO APPS SCRIPT
    // ======================================

    const response =

      await fetch(

        requestUrl,

        {

          method: "GET"

        }

      );


    // อ่านผลจาก Apps Script

    const result =

      await response.text();


    console.log(

      "Apps Script response:",

      result

    );


    // ======================================
    // CHECK RESULT
    // ======================================

    if (

      !result.startsWith("SUCCESS")

    ) {

      throw new Error(result);

    }


    // ======================================
    // SUCCESS
    // ======================================

    showResult(

      "เชื่อมบัญชีสำเร็จ 🎉",

      "success"

    );


    button.innerText =

      "เชื่อมบัญชีสำเร็จ";


  } catch (error) {


    console.error(error);


    button.disabled = false;


    button.innerText =

      "ยืนยันข้อมูล";


    showResult(

      "ไม่สามารถบันทึกข้อมูลได้ กรุณาลองใหม่",

      "error"

    );

  }

}


// ==========================================
// SHOW RESULT
// ==========================================

function showResult(

  message,

  type

) {

  const result =

    document.getElementById(

      "result"

    );


  result.innerText =

    message;


  result.className =

    type;

}


// ==========================================
// START LIFF
// ==========================================

main();

</script>


</body>

</html>
