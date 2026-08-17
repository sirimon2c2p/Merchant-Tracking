<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <title>Merchant Tracking</title>

  <script src="https://static.line-scdn.net/liff/edge/2/sdk.js"></script>

  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 30px 20px;
      background: #ffffff;
      text-align: center;
    }

    .container {
      max-width: 500px;
      margin: auto;
    }

    h1 {
      font-size: 24px;
      margin-bottom: 10px;
    }

    p {
      color: #666;
      line-height: 1.6;
    }

    button {
      width: 100%;
      padding: 14px;
      margin-top: 20px;
      border: none;
      border-radius: 8px;
      background: #06c755;
      color: white;
      font-size: 16px;
      cursor: pointer;
    }

    #status {
      margin-top: 20px;
      font-size: 14px;
    }
  </style>
</head>

<body>

<div class="container">

  <h1>Merchant Tracking</h1>

  <p id="message">
    กำลังตรวจสอบข้อมูล...
  </p>

  <button id="continueButton" style="display:none;">
    ดำเนินการต่อ
  </button>

  <div id="status"></div>

</div>


<script>

  // ==========================================
  // CONFIG
  // ==========================================

  const LIFF_ID = "2011137760-Da2862OU";

  // เอา URL Web App ของ Google Apps Script มาใส่ตรงนี้
  const GOOGLE_SCRIPT_URL = "ใส่_GOOGLE_APPS_SCRIPT_WEB_APP_URL";


  // ==========================================
  // INITIALIZE LIFF
  // ==========================================

  async function main() {

    try {

      await liff.init({
        liffId: LIFF_ID
      });

      console.log("LIFF initialized");


      // ถ้ายังไม่ได้ login
      if (!liff.isLoggedIn()) {

        liff.login();

        return;
      }


      // ==========================================
      // GET LINE PROFILE
      // ==========================================

      const profile = await liff.getProfile();

      console.log("LINE Profile:", profile);


      const lineUserId = profile.userId;
      const displayName = profile.displayName;


      // ==========================================
      // CHECK FRIENDSHIP
      // ==========================================

      let friendFlag = false;

      try {

        const friendship = await liff.getFriendship();

        friendFlag = friendship.friendFlag;

      } catch (error) {

        console.log("Friendship check failed:", error);

      }


      // ==========================================
      // GET MERCHANT ID FROM URL
      // ==========================================

      const params = new URLSearchParams(window.location.search);

      const merchantId = params.get("merchantId") || "";


      // ==========================================
      // SEND TRACKING DATA
      // ==========================================

      const trackingData = {

        lineUserId: lineUserId,

        displayName: displayName,

        merchantId: merchantId,

        friendFlag: friendFlag,

        status: "unlocked"

      };


      console.log("Tracking Data:", trackingData);


      await sendTrackingData(trackingData);


      // ==========================================
      // SHOW SUCCESS
      // ==========================================

      document.getElementById("message").innerText =
        "บันทึกข้อมูลเรียบร้อยแล้ว";

      document.getElementById("continueButton").style.display =
        "block";


      // ==========================================
      // CONTINUE BUTTON
      // ==========================================

      document.getElementById("continueButton").onclick = function() {

        // ใส่ URL ปลายทางของ Merchant App ตรงนี้
        window.location.href = "https://YOUR-MERCHANT-APP-URL";

      };


    } catch (error) {

      console.error(error);

      document.getElementById("message").innerText =
        "เกิดข้อผิดพลาด กรุณาลองใหม่อีกครั้ง";

      document.getElementById("status").innerText =
        error.message;

    }

  }


  // ==========================================
  // SEND DATA TO GOOGLE APPS SCRIPT
  // ==========================================

  async function sendTrackingData(data) {

    try {

      await fetch(GOOGLE_SCRIPT_URL, {

        method: "POST",

        mode: "no-cors",

        headers: {

          "Content-Type": "text/plain;charset=utf-8"

        },

        body: JSON.stringify(data)

      });

      console.log("Tracking sent");

    } catch (error) {

      console.error("Tracking error:", error);

    }

  }


  main();

</script>

</body>
</html>
