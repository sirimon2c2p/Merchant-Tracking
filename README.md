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
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 40px 20px;
      text-align: center;
      background: #ffffff;
    }

    .container {
      max-width: 500px;
      margin: auto;
    }

    h1 {
      font-size: 24px;
      margin-bottom: 15px;
    }

    #message {
      color: #555;
      line-height: 1.6;
    }

    #status {
      margin-top: 20px;
      font-size: 14px;
      color: #888;
    }

    .success {
      color: #06c755;
    }

    .error {
      color: #d93025;
    }
  </style>

</head>


<body>

  <div class="container">

    <h1>Merchant Tracking</h1>

    <p id="message">
      กำลังตรวจสอบข้อมูล...
    </p>

    <div id="status"></div>

  </div>


  <script>

    // ==================================================
    // CONFIG
    // ==================================================

    const LIFF_ID = "2011137760-Da2862OU";

    const GOOGLE_SCRIPT_URL =
      "https://script.google.com/macros/s/AKfycbxcuzMhNbehlk_ce2pejXjMqkKcL2CJJgjAcz6r08emaGLSGb48AvQg4gLZOH3Glchr/exec";


    // ==================================================
    // MAIN
    // ==================================================

    async function main() {

      try {

        showMessage(
          "กำลังเชื่อมต่อกับ LINE...",
          ""
        );


        // ------------------------------------------------
        // 1. Initialize LIFF
        // ------------------------------------------------

        await liff.init({
          liffId: LIFF_ID
        });


        console.log("LIFF initialized");


        // ------------------------------------------------
        // 2. Check Login
        // ------------------------------------------------

        if (!liff.isLoggedIn()) {

          showMessage(
            "กำลังเข้าสู่ระบบ LINE...",
            ""
          );

          liff.login();

          return;
        }


        // ------------------------------------------------
        // 3. Get LINE Profile
        // ------------------------------------------------

        const profile = await liff.getProfile();

        console.log("LINE Profile:", profile);


        const lineUserId =
          profile.userId || "";

        const displayName =
          profile.displayName || "";


        // ------------------------------------------------
        // 4. Check Friend Status
        // ------------------------------------------------

        let friendFlag = false;

        try {

          const friendship =
            await liff.getFriendship();

          friendFlag =
            friendship.friendFlag === true;

          console.log(
            "Friend Status:",
            friendFlag
          );

        } catch (friendError) {

          console.log(
            "Friendship check error:",
            friendError
          );

        }


        // ------------------------------------------------
        // 5. Get Merchant ID from URL
        // ------------------------------------------------

        const urlParams =
          new URLSearchParams(
            window.location.search
          );


        const merchantId =
          urlParams.get("merchantId") || "";


        console.log(
          "Merchant ID:",
          merchantId
        );


        // ------------------------------------------------
        // 6. Prepare Data
        // ------------------------------------------------

        const trackingData = {

          lineUserId: lineUserId,

          displayName: displayName,

          merchantId: merchantId,

          friendFlag: friendFlag,

          status: "unlocked"

        };


        console.log(
          "Tracking Data:",
          trackingData
        );


        // ------------------------------------------------
        // 7. Send Data to Google Apps Script
        // ------------------------------------------------

        await sendToGoogleSheet(
          trackingData
        );


        // ------------------------------------------------
        // 8. Show Success
        // ------------------------------------------------

        showMessage(
          "ปลดล็อกสำเร็จ",
          "success"
        );

        document.getElementById("status")
          .innerText =
          "บันทึกข้อมูลเรียบร้อยแล้ว";


      } catch (error) {

        console.error(error);

        showMessage(
          "เกิดข้อผิดพลาด",
          "error"
        );

        document.getElementById("status")
          .innerText =
          error.message || "Unknown error";

      }

    }


    // ==================================================
    // SEND DATA TO GOOGLE APPS SCRIPT
    // ==================================================

    async function sendToGoogleSheet(data) {

      try {

        await fetch(
          GOOGLE_SCRIPT_URL,
          {
            method: "POST",

            mode: "no-cors",

            headers: {
              "Content-Type":
                "text/plain;charset=utf-8"
            },

            body: JSON.stringify(data)
          }
        );


        console.log(
          "Data sent to Google Apps Script"
        );


      } catch (error) {

        console.error(
          "Send data error:",
          error
        );

        throw error;

      }

    }


    // ==================================================
    // SHOW MESSAGE
    // ==================================================

    function showMessage(
      message,
      type
    ) {

      const messageElement =
        document.getElementById(
          "message"
        );

      messageElement.innerText =
        message;


      messageElement.className =
        type;

    }


    // ==================================================
    // START
    // ==================================================

    main();

  </script>

</body>

</html>
