<!DOCTYPE html>
<html lang="th">

<head>
  <meta charset="UTF-8">

  <meta
    name="viewport"
    content="width=device-width, initial-scale=1.0"
  >

  <title>Merchant Tracking</title>

  <!-- LINE LIFF SDK -->
  <script src="https://static.line-scdn.net/liff/edge/2/sdk.js"></script>

  <style>

    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      padding: 20px;

      min-height: 100vh;

      background: #f7f7f7;

      font-family:
        -apple-system,
        BlinkMacSystemFont,
        "Segoe UI",
        Arial,
        sans-serif;

      display: flex;
      align-items: center;
      justify-content: center;
    }

    .container {
      width: 100%;
      max-width: 400px;
    }

    .card {
      background: #ffffff;

      border-radius: 16px;

      padding: 30px 24px;

      box-shadow:
        0 4px 20px rgba(0, 0, 0, 0.08);
    }

    h1 {
      margin: 0 0 10px;

      text-align: center;

      font-size: 24px;

      font-weight: 700;

      color: #222;
    }

    .subtitle {
      margin: 0 0 28px;

      text-align: center;

      font-size: 15px;

      line-height: 1.6;

      color: #666;
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

      height: 48px;

      padding: 0 14px;

      border: 1px solid #d9d9d9;

      border-radius: 10px;

      font-size: 16px;

      outline: none;

      background: #fff;
    }

    input:focus {
      border-color: #06C755;
    }

    input::placeholder {
      color: #aaa;
    }

    button {
      width: 100%;

      height: 48px;

      border: 0;

      border-radius: 10px;

      background: #06C755;

      color: white;

      font-size: 16px;

      font-weight: 600;

      cursor: pointer;
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

      width: 16px;
      height: 16px;

      margin-right: 7px;

      vertical-align: -3px;

      border: 2px solid #ddd;

      border-top-color: #06C755;

      border-radius: 50%;

      animation: spin 0.8s linear infinite;
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

      background: #eefaf2;

      color: #16833a;
    }

    #message.error {
      display: block;

      background: #fff1f1;

      color: #c62828;
    }

  </style>

</head>


<body>

  <div class="container">

    <div class="card">

      <h1>
        เชื่อมบัญชีร้านค้า
      </h1>

      <p class="subtitle">
        กรุณากรอก Merchant ID<br>
        เพื่อเชื่อมบัญชีร้านค้าของคุณกับ LINE OA
      </p>


      <div id="formArea">

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
          type="button"
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

    // =====================================================
    // CONFIG
    // =====================================================

    const LIFF_ID =
      "2011658045-4N77aswc";


    /*
      สำคัญ:
      ใส่ Google Apps Script Web App URL
      ที่ลงท้ายด้วย /exec
    */

    const GOOGLE_SCRIPT_URL =
      "https://script.google.com/macros/s/AKfycbyVWsp7-qk-XZXs5xRrjrVneqnGVzM-rRnERp4RPm8TPws0hd1pioTcCKCWu2AU06e2/exec";


    // =====================================================
    // LIFF INIT
    // =====================================================

    async function initLIFF() {

      try {

        await liff.init({
          liffId: LIFF_ID
        });

        console.log("LIFF initialized");


        /*
          ถ้าเปิดจาก Browser ปกติ
          และยังไม่ได้ Login
        */

        if (!liff.isLoggedIn()) {

          if (!liff.isInClient()) {

            liff.login();

            return;
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


    // =====================================================
    // SUBMIT
    // =====================================================

    async function submitForm() {

      const merchantIdInput =
        document.getElementById("merchantId");

      const submitBtn =
        document.getElementById("submitBtn");

      const loading =
        document.getElementById("loading");


      const merchantId =
        merchantIdInput.value.trim();


      // -----------------------------
      // ตรวจ Merchant ID
      // -----------------------------

      if (!merchantId) {

        showError(
          "กรุณากรอก Merchant ID"
        );

        merchantIdInput.focus();

        return;
      }


      try {

        submitBtn.disabled = true;

        loading.style.display = "block";

        hideMessage();


        // -----------------------------
        // ตรวจ Login
        // -----------------------------

        if (!liff.isLoggedIn()) {

          if (!liff.isInClient()) {

            liff.login();

            return;

          }

          throw new Error(
            "ไม่พบการเข้าสู่ระบบ LINE"
          );

        }


        // -----------------------------
        // ดึง LINE Profile
        // -----------------------------

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


        // -----------------------------
        // เตรียมข้อมูล
        // -----------------------------

        const data = {

          lineUserId:
            profile.userId,

          displayName:
            profile.displayName,

          merchantId:
            merchantId

        };


        console.log(
          "Sending:",
          data
        );


        // =================================================
        // ส่งข้อมูลไป Google Apps Script
        //
        // ใช้ POST + URLSearchParams
        // ไม่มี JSON Content-Type
        // เพื่อหลีกเลี่ยง CORS preflight
        // =================================================

        const formData =
          new URLSearchParams();

        formData.append(
          "data",
          JSON.stringify(data)
        );


        const response =
          await fetch(
            GOOGLE_SCRIPT_URL,
            {
              method: "POST",

              body: formData
            }
          );


        console.log(
          "Response status:",
          response.status
        );


        // -----------------------------
        // สำเร็จ
        // -----------------------------

        loading.style.display = "none";


        document
          .getElementById("formArea")
          .style.display = "none";


        showSuccess(
          "เชื่อมบัญชีสำเร็จ<br>" +
          "Merchant ID ถูกบันทึกเรียบร้อยแล้ว"
        );


      } catch (error) {

        console.error(
          "Submit error:",
          error
        );


        loading.style.display = "none";

        submitBtn.disabled = false;


        showError(
          "ไม่สามารถบันทึกข้อมูลได้<br>" +
          "กรุณาลองใหม่อีกครั้ง"
        );

      }

    }


    // =====================================================
    // MESSAGE
    // =====================================================

    function showSuccess(text) {

      const message =
        document.getElementById("message");

      message.innerHTML = text;

      message.className = "success";

      message.style.display = "block";

    }


    function showError(text) {

      const message =
        document.getElementById("message");

      message.innerHTML = text;

      message.className = "error";

      message.style.display = "block";

    }


    function hideMessage() {

      const message =
        document.getElementById("message");

      message.style.display = "none";

    }


    // =====================================================
    // START
    // =====================================================

    initLIFF();

  </script>

</body>

</html>
