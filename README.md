<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <title>Merchant Tracking</title>

  <script src="https://static.line-scdn.net/liff/edge/2/sdk.js"></script>

  <style>
    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      padding: 24px;
      min-height: 100vh;
      background: #f7f7f7;
      font-family: Arial, sans-serif;

      display: flex;
      justify-content: center;
      align-items: center;
    }

    .container {
      width: 100%;
      max-width: 420px;
    }

    .card {
      background: #ffffff;
      border-radius: 18px;
      padding: 32px 24px;
      box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
    }

    h1 {
      margin: 0 0 10px;
      text-align: center;
      font-size: 24px;
      color: #222;
    }

    .subtitle {
      margin: 0 0 28px;
      text-align: center;
      color: #666;
      font-size: 15px;
      line-height: 1.6;
    }

    label {
      display: block;
      margin-bottom: 8px;
      font-size: 14px;
      font-weight: bold;
      color: #333;
    }

    input {
      width: 100%;
      height: 48px;
      padding: 0 14px;
      border: 1px solid #ddd;
      border-radius: 10px;
      font-size: 16px;
      outline: none;
    }

    input:focus {
      border-color: #06C755;
    }

    button {
      width: 100%;
      height: 48px;
      margin-top: 20px;
      border: none;
      border-radius: 10px;
      background: #06C755;
      color: #fff;
      font-size: 16px;
      font-weight: bold;
      cursor: pointer;
    }

    button:disabled {
      opacity: 0.6;
      cursor: default;
    }

    .loading {
      display: none;
      text-align: center;
      margin-top: 20px;
      color: #666;
      font-size: 14px;
    }

    .message {
      display: none;
      text-align: center;
      margin-top: 20px;
      line-height: 1.6;
      font-size: 15px;
    }

    .success {
      color: #06C755;
    }

    .error {
      color: #d93025;
    }
  </style>
</head>

<body>

  <div class="container">

    <div class="card">

      <h1>Merchant Tracking</h1>

      <p class="subtitle">
        กรุณากรอก Merchant ID<br>
        เพื่อเชื่อมบัญชีร้านค้ากับ LINE OA
      </p>

      <label for="merchantId">Merchant ID</label>

      <input
        type="text"
        id="merchantId"
        placeholder="กรอก Merchant ID"
        autocomplete="off"
      >

      <button id="submitBtn" onclick="submitForm()">
        ยืนยันข้อมูล
      </button>

      <div id="loading" class="loading">
        🔄 กำลังบันทึกข้อมูล...
      </div>

      <div id="message" class="message"></div>

    </div>

  </div>


  <script>

    const LIFF_ID = "2011658045-4N77aswc";

    // ใส่ Google Apps Script Web App URL /exec ของเรา
    const GOOGLE_SCRIPT_URL = "https://script.google.com/macros/s/AKfycbyVWsp7-qk-XZXs5xRrjrVneqnGVzM-rRnERp4RPm8TPws0hd1pioTcCKCWu2AU06e2/exec";


    async function initLIFF() {

      try {

        await liff.init({
          liffId: LIFF_ID
        });

        if (!liff.isLoggedIn()) {

          if (liff.isInClient()) {
            return;
          }

          liff.login();
          return;
        }

      } catch (error) {

        console.error("LIFF Error:", error);

        showMessage(
          "ไม่สามารถเชื่อมต่อกับ LINE ได้<br>กรุณาลองใหม่อีกครั้ง",
          "error"
        );

      }

    }


    async function submitForm() {

      const merchantId =
        document.getElementById("merchantId").value.trim();

      const submitBtn =
        document.getElementById("submitBtn");

      const loading =
        document.getElementById("loading");


      if (!merchantId) {

        showMessage(
          "กรุณากรอก Merchant ID",
          "error"
        );

        return;
      }


      try {

        submitBtn.disabled = true;

        loading.style.display = "block";

        document.getElementById("message").style.display = "none";


        if (!liff.isLoggedIn()) {

          if (!liff.isInClient()) {
            liff.login();
            return;
          }

          throw new Error("ยังไม่ได้เข้าสู่ระบบ LINE");

        }


        const profile = await liff.getProfile();

        let friendFlag = false;

        try {

          const friendship =
            await liff.getFriendship();

          friendFlag = friendship.friendFlag;

        } catch (e) {

          console.log("Friendship check skipped");

        }


        const data = {

          lineUserId: profile.userId,

          displayName: profile.displayName,

          merchantId: merchantId,

          friendFlag: friendFlag,

          status: "unlocked"

        };


        await fetch(GOOGLE_SCRIPT_URL, {

          method: "POST",

          mode: "no-cors",

          headers: {
            "Content-Type": "application/json"
          },

          body: JSON.stringify(data)

        });


        loading.style.display = "none";

        document.getElementById("merchantId").style.display = "none";

        document.querySelector("label").style.display = "none";

        submitBtn.style.display = "none";


        showMessage(
          "✅ เชื่อมบัญชีสำเร็จ<br>Merchant ID ถูกบันทึกเรียบร้อยแล้ว",
          "success"
        );


      } catch (error) {

        console.error(error);

        loading.style.display = "none";

        submitBtn.disabled = false;

        showMessage(
          "เกิดข้อผิดพลาด<br>กรุณาลองใหม่อีกครั้ง",
          "error"
        );

      }

    }


    function showMessage(text, type) {

      const message =
        document.getElementById("message");

      message.innerHTML = text;

      message.className =
        "message " + type;

      message.style.display = "block";

    }


    initLIFF();

  </script>

</body>
</html>
