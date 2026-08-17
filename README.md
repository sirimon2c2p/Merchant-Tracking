<!DOCTYPE html>

<html>

<head>

  <meta charset="UTF-8">

  <meta
    name="viewport"
    content="width=device-width, initial-scale=1.0"
  >

  <title>
    Merchant Verification
  </title>


  <script
    src="https://static.line-scdn.net/liff/edge/2/sdk.js">
  </script>


</head>


<body>


  <h2>
    ยืนยันข้อมูลร้านค้า
  </h2>


  <p>
    LINE ID
  </p>

  <input
    type="text"
    id="lineId"
    placeholder="เช่น ging2541"
  >


  <p>
    Merchant ID
  </p>

  <input
    type="text"
    id="merchantId"
    placeholder="กรอก Merchant ID"
  >


  <br><br>


  <button
    onclick="submitForm()"
  >
    ยืนยัน
  </button>


  <p id="result"></p>


<script>


const LIFF_ID =
  "2011132851-VrQEJ6SP";


const WEB_APP_URL =
  "https://script.google.com/macros/s/AKfycbyKQ7qpjq-XB4P2FgFYs7YvYNnnJ5Cr-_3scRK3Fs_6w2JwqkEulE27BRrVdEBJRBRN/exec";


// ============================
// INITIALIZE LIFF
// ============================

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

    document.getElementById(
      "result"
    ).innerText =
      "ไม่สามารถเชื่อมต่อ LINE ได้";

  }

}


// ============================
// SUBMIT FORM
// ============================

async function submitForm() {

  const lineId =
    document.getElementById(
      "lineId"
    ).value.trim();


  const merchantId =
    document.getElementById(
      "merchantId"
    ).value.trim();


  // ============================
  // CHECK LINE ID
  // ============================

  if (!lineId) {

    document.getElementById(
      "result"
    ).innerText =
      "กรุณากรอก LINE ID";

    return;

  }


  // ============================
  // CHECK MERCHANT ID
  // ============================

  if (merchantId.length !== 10) {

    document.getElementById(
      "result"
    ).innerText =
      "กรุณากรอก Merchant ID ให้ครบ 10 ตัว";

    return;

  }


  try {


    // ============================
    // GET LINE PROFILE
    // ============================

    const profile =
      await liff.getProfile();


    const userId =
      profile.userId;


    const displayName =
      profile.displayName;


    // ============================
    // SEND TO APPS SCRIPT
    // ============================

    const formData =
      new URLSearchParams();


    formData.append(
      "userId",
      userId
    );


    formData.append(
      "lineId",
      lineId
    );


    formData.append(
      "displayName",
      displayName
    );


    formData.append(
      "merchantId",
      merchantId
    );


    document.getElementById(
      "result"
    ).innerText =
      "กำลังบันทึกข้อมูล...";


    await fetch(
      WEB_APP_URL,
      {

        method: "POST",

        mode: "no-cors",

        headers: {

          "Content-Type":
            "application/x-www-form-urlencoded"

        },

        body:
          formData.toString()

      }
    );


    document.getElementById(
      "result"
    ).innerText =
      "เชื่อมต่อสำเร็จ 🎉";


  } catch (error) {

    console.error(error);

    document.getElementById(
      "result"
    ).innerText =
      "เกิดข้อผิดพลาด กรุณาลองใหม่";

  }

}


main();


</script>


</body>

</html>
