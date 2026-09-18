<!DOCTYPE html>
<html lang="th">

<head>

  <meta charset="UTF-8">

  <meta
    name="viewport"
    content="width=device-width, initial-scale=1.0"
  >

  <title>Merchant Tracking</title>

  <script src="https://static.line-scdn.net/liff/edge/2/sdk.js"></script>


  <style>

    * {
      box-sizing: border-box;
    }


    body {
      margin: 0;
      padding: 20px;

      min-height: 100vh;

      background: #f6f7f8;

      font-family:
        -apple-system,
        BlinkMacSystemFont,
        "Segoe UI",
        Arial,
        sans-serif;

      display: flex;
      justify-content: center;
      align-items: center;
    }


    .wrapper {
      width: 100%;
      max-width: 420px;
    }


    .card {
      background: #ffffff;

      border-radius: 20px;

      padding: 32px 24px;

      box-shadow:
        0 6px 24px
        rgba(0, 0, 0, 0.08);
    }


    .logo {
      width: 64px;
      height: 64px;

      margin: 0 auto 20px;

      border-radius: 16px;

      background: #06c755;

      display: flex;
      justify-content: center;
      align-items: center;

      font-size: 30px;
    }


    h1 {
      margin: 0;

      text-align: center;

      font-size: 24px;

      color: #222;
    }


    .description {
      margin: 10px 0 28px;

      text-align: center;

      color: #666;

      font-size: 15px;

      line-height: 1.6;
    }


    .form-group {
      margin-bottom: 18px;
    }


    label {
      display: block;

      margin-bottom: 8px;

      font-size: 14px;

      font-weight: 600;

      color: #333;
    }


    input {
      width: 100%;

      height: 50px;

      padding: 0 15px;

      border:
        1px solid #dcdcdc;

      border-radius: 10px;

      background: #fff;

      font-size: 16px;

      color: #222;

      outline: none;

      transition:
        border-color 0.2s;
    }


    input:focus {
      border-color: #06c755;
    }


    input::placeholder {
      color: #aaa;
    }


    button {
      width: 100%;

      height: 50px;

      border: 0;

      border-radius: 10px;

      background: #06c755;

      color: #fff;

      font-size: 16px;

      font-weight: 600;

      cursor: pointer;
    }


    button:active {
      transform: scale(0.99);
    }


    button:disabled {
      opacity: 0.6;

      cursor: not-allowed;
    }


    #loading {
      display: none;

      margin-top: 18px;

      text-align: center;

      color: #666;

      font-size: 14px;
    }


    .spinner {
      display: inline-block;

      width: 18px;
      height: 18px;

      margin-right: 7px;

      vertical-align: -4px;

      border:
        2px solid #ddd;

      border-top-color:
        #06c755;

      border-radius: 50%;

      animation:
        spin 0.8s linear infinite;
    }


    @keyframes spin {

      to {
        transform: rotate(360deg);
      }

    }


    #message {
      display: none;

      margin-top: 18px;

      padding: 14px;

      border-radius: 10px;

      text-align: center;

      font-size: 14px;

      line-height: 1.6;
    }


    #message.success {
      display: block;

      background: #eefbf3;

      color: #16833a;
    }


    #message.error {
      display: block;

      background: #fff1f1;

      color: #c62828;
    }


    .hidden {
      display: none !important;
    }

  </style>

</head>


<body>


  <div class="wrapper">

    <div class="card">


      <div class="logo">
        💳
      </div>


      <h1>
        เชื่อมบัญชีร้านค้า
      </h1>


      <p class="description">
        กรุณากรอก Merchant ID<br>
        เพื่อเชื่อมบัญชีร้านค้าของคุณกับ LINE OA
      </p>


      <div
        id="formArea"
      >

        <div class="form-group">

          <label for="merchantId">
            Merchant ID
          </label>

          <input
            type="text"
            id="merchantId"
            placeholder="กรอก Merchant ID"
            autocomplete="off"
          >

        </div>


        <button
          id="submitBtn"
          onclick="submitForm()"
        >
          ยืนยันข้อมูล
        </button>


        <div id="loading">

          <span class="spinner"></span>

          กำลังบันทึกข้อมูล...

        </div>


      </div>


      <div id="message"></div>


    </div>

  </div>



  <script>


    // ==================================================
    // SETTINGS
    // ==================================================

    const LIFF_ID =
      "2011658045-4N77aswc";


    // ⚠️ ใส่ Apps Script /exec URL ของคุณตรงนี้
    const GOOGLE_SCRIPT_URL =
      "https://script.google.com/macros/s/AKfycbyVWsp7-qk-XZXs5xRrjrVneqnGVzM-rRnERp4RPm8TPws0hd1pioTcCKCWu2AU06e2/exec";


    // ==================================================
    // INIT LIFF
    // ==================================================

    async function initLIFF() {

      try {

        await liff.init({
          liffId: LIFF_ID
        });


        console.log(
          "LIFF initialized"
        );


        // ถ้าเปิดจาก browser และยังไม่ได้ login
        if (!liff.isLoggedIn()) {

          if (!liff.isInClient()) {

            liff.login();

          }

        }


      } catch (error) {

        console.error(
          "LIFF init error:",
          error
        );


        showError(
          "ไม่สามารถเชื่อมต่อกับ LINE ได้<br>" +
          "กรุณาลองใหม่อีกครั้ง"
        );

      }

    }


    // ==================================================
    // SUBMIT
    // ==================================================

    async function submitForm() {


      const merchantIdInput =
        document.getElementById(
          "merchantId"
        );


      const submitButton =
        document.getElementById(
          "submitBtn"
        );


      const loading =
        document.getElementById(
          "loading"
        );


      const merchantId =
        merchantIdInput.value.trim();


      // ------------------------------
      // ตรวจ Merchant ID
      // ------------------------------

      if (!merchantId) {

        showError(
          "กรุณากรอก Merchant ID"
        );

        merchantIdInput.focus();

        return;

      }


      // ------------------------------
      // Loading
      // ------------------------------

      submitButton.disabled = true;

      loading.style.display =
        "block";

      hideMessage();


      try {


        // ------------------------------
        // ตรวจ LIFF Login
        // ------------------------------

        if (!liff.isLoggedIn()) {

          if (!liff.isInClient()) {

            liff.login();

            return;

          }

          throw new Error(
            "ไม่พบการเข้าสู่ระบบ LINE"
          );

        }


        // ------------------------------
        // ดึง Profile
        // ------------------------------

        const profile =
          await liff.getProfile();


        console.log(
          "LINE User ID:",
          profile.userId
        );


        console.log(
          "Display Name:",
          profile.displayName
        );


        // ------------------------------
        // เตรียมข้อมูล
        // ------------------------------

        const data = {

          lineUserId:
            profile.userId,

          displayName:
            profile.displayName,

          merchantId:
            merchantId

        };


        console.log(
          "Sending data:",
          data
        );


        // ------------------------------
        // ส่ง Google Apps Script
        //
        // ใช้ URLSearchParams
        // เพื่อไม่ให้เกิด CORS preflight
        // ------------------------------

        const formData =
          new URLSearchParams();


        formData.append(
          "data",
          JSON.stringify(data)
        );


        await fetch(
          GOOGLE_SCRIPT_URL,
          {
            method: "POST",

            body: formData
          }
        );


        // ------------------------------
        // แสดง Success
        // ------------------------------

        loading.style.display =
          "none";


        document
          .getElementById("formArea")
          .classList.add("hidden");


        showSuccess(
          "เชื่อมบัญชีสำเร็จ<br>" +
          "Merchant ID ถูกบันทึกเรียบร้อยแล้ว"
        );


        console.log(
          "Data sent successfully"
        );


      } catch (error) {


        console.error(
          "Submit error:",
          error
        );


        loading.style.display =
          "none";


        submitButton.disabled =
          false;


        showError(
          "ไม่สามารถบันทึกข้อมูลได้<br>" +
          "กรุณาลองใหม่อีกครั้ง"
        );

      }

    }


    // ==================================================
    // MESSAGE
    // ==================================================

    function showSuccess(text) {

      const message =
        document.getElementById(
          "message"
        );


      message.innerHTML =
        text;


      message.className =
        "success";


      message.style.display =
        "block";

    }


    function showError(text) {

      const message =
        document.getElementById(
          "message"
        );


      message.innerHTML =
        text;


      message.className =
        "error";


      message.style.display =
        "block";

    }


    function hideMessage() {

      const message =
        document.getElementById(
          "message"
        );


      message.style.display =
        "none";

    }


    // ==================================================
    // START
    // ==================================================

    initLIFF();


  </script>


</body>

</html>
