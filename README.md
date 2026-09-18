<script>

const LIFF_ID = "2011658045-4N77aswc";

const GOOGLE_SCRIPT_URL =
  "https://script.google.com/macros/s/AKfycbyVWsp7-qk-XZXs5xRrjrVneqnGVzM-rRnERp4RPm8TPws0hd1pioTcCKCWu2AU06e2/exec";


async function initLIFF() {

  try {

    await liff.init({
      liffId: LIFF_ID
    });

    console.log("LIFF initialized");

    if (!liff.isLoggedIn()) {

      if (!liff.isInClient()) {

        liff.login();
        return;

      }

      return;
    }

  } catch (error) {

    console.error("LIFF initialization error:", error);

    showMessage(
      "ไม่สามารถเชื่อมต่อกับ LINE ได้<br>กรุณาลองใหม่อีกครั้ง",
      "error"
    );

  }

}


async function submitForm() {

  const merchantInput =
    document.getElementById("merchantId");

  const submitBtn =
    document.getElementById("submitBtn");

  const loading =
    document.getElementById("loading");

  const merchantId =
    merchantInput.value.trim();


  // ตรวจ Merchant ID
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


    // --------------------------------
    // ตรวจ LIFF Login
    // --------------------------------

    if (!liff.isLoggedIn()) {

      if (!liff.isInClient()) {

        liff.login();
        return;

      }

      throw new Error(
        "ยังไม่ได้เข้าสู่ระบบ LINE"
      );

    }


    // --------------------------------
    // ดึง LINE Profile
    // --------------------------------

    const profile =
      await liff.getProfile();


    console.log("LINE User ID:", profile.userId);

    console.log(
      "Display Name:",
      profile.displayName
    );


    // --------------------------------
    // เตรียมข้อมูล
    // --------------------------------

    const data = {

      lineUserId: profile.userId,

      displayName: profile.displayName,

      merchantId: merchantId

    };


    console.log(
      "Data to send:",
      data
    );


    // --------------------------------
    // ส่งไป Google Apps Script
    //
    // ใช้ URLSearchParams
    // เพื่อไม่ให้เกิด CORS preflight
    // --------------------------------

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
      "Apps Script response:",
      response
    );


    // --------------------------------
    // สำเร็จ
    // --------------------------------

    loading.style.display = "none";

    merchantInput.style.display = "none";

    document.querySelector("label").style.display = "none";

    submitBtn.style.display = "none";


    showMessage(
      "✅ เชื่อมบัญชีสำเร็จ<br>Merchant ID ถูกบันทึกเรียบร้อยแล้ว",
      "success"
    );


  } catch (error) {

    console.error(
      "Submit error:",
      error
    );


    loading.style.display = "none";

    submitBtn.disabled = false;


    showMessage(
      "เกิดข้อผิดพลาด<br>ไม่สามารถบันทึกข้อมูลได้",
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
