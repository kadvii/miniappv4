<script>
  import "https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4";
  import "https://cdn.marmot-cloud.com/npm/hylid-bridge/2.10.0/index.js";

  let authCode = $state("");
  let token = $state("");

  function auth() {
    console.log("Auth button clicked"); // Debug log

    if (typeof my === "undefined") {
      alert(
        "Error: Hylid bridge (my) is not loaded. Make sure you're running this in a mini app environment.",
      );
      return;
    }

    my.getAuthCode({
      scopes: ["auth_base", "USER_ID"],
      success: (res) => {
        authCode = res.authCode;
        console.log("Auth code received:", authCode);

        fetch("https://its.mouamle.space/api/auth-with-superQi", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            token: authCode,
          }),
        })
          .then((res) => res.json())
          .then((data) => {
            token = data.token;
            console.log("Token received:", token);
            my.alert({
              content: "Login successful",
            });
          })
          .catch((err) => {
            console.error("Auth error:", err);
            let errorDetails = "";
            if (err && typeof err === "object") {
              errorDetails = JSON.stringify(err, null, 2);
            } else {
              errorDetails = String(err);
            }
            my.alert({
              content: "Error: " + errorDetails,
            });
          });
      },
      fail: (res) => {
        console.error("Auth failed:", res.authErrorScopes);
        my.alert({
          content:
            "Authentication failed: " + JSON.stringify(res.authErrorScopes),
        });
      },
    });
  }

  function pay() {
    if (!token) {
      my.alert({
        content: "Please authenticate first",
      });
      return;
    }

    fetch("https://its.mouamle.space/api/payment", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: token,
      },
    })
      .then((res) => res.json())
      .then((data) => {
        my.tradePay({
          paymentUrl: data.url,
          success: (res) => {
            my.alert({
              content: "Payment successful",
            });
          },
        });
      })
      .catch((err) => {
        console.error("Payment error:", err);
        my.alert({
          content: "Payment failed: " + String(err),
        });
      });
  }

  function copyAuthCode() {
    if (!authCode) {
      my.alert({
        content: "No auth code to copy",
      });
      return;
    }
    navigator.clipboard.writeText(authCode);
    my.alert({
      content: "Auth code copied!",
    });
  }

  function scan() {
    console.log("Scan button clicked"); // Debug log

    if (typeof my === "undefined") {
      alert(
        "Error: Hylid bridge (my) is not loaded. Make sure you're running this in a mini app environment.",
      );
      return;
    }

    my.scan({
      type: "qr",
      success: (res) => {
        console.log("QR code scanned:", res.code);
        my.alert({
          title: "Scanned Code",
          content: res.code,
        });
      },
      fail: (err) => {
        console.error("Scan failed:", err);
        my.alert({
          title: "Scan Failed",
          content: "Failed to scan QR code",
        });
      },
    });
  }
</script>

<main class="min-h-screen bg-black flex items-center justify-center">
  <div class="flex flex-col gap-4">
    <button
      class="bg-blue-500 hover:bg-blue-600 text-white px-6 py-3 rounded-lg text-lg font-semibold transition shadow-lg"
      onclick={auth}
    >
      Auth
    </button>
    <button
      class="bg-green-500 hover:bg-green-600 text-white px-6 py-3 rounded-lg text-lg font-semibold transition shadow-lg"
      onclick={scan}
    >
      Scan QR Code
    </button>
  </div>
</main>
