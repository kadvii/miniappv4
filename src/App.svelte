<script>
  import "https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4";
  import "https://cdn.marmot-cloud.com/npm/hylid-bridge/2.10.0/index.js";

  var authCode = "";
  var token = "";

  function auth() {
    my.getAuthCode({
      scopes: ["auth_base", "USER_ID"],
      success: (res) => {
        authCode = res.authCode;
        document.getElementById("authCode").textContent = authCode;

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
            my.alert({
              content: "Login successful",
            });
          })
          .catch((err) => {
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
        console.log(res.authErrorScopes);
      },
    });
  }

  function pay() {
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
        my.alert({
          content: "Payment failed",
        });
      });
  }

  function copyAuthCode() {
    navigator.clipboard.writeText(authCode);
  }
</script>

<main class="min-h-screen bg-black flex items-center justify-center">
  <button
    class="bg-blue-500 hover:bg-blue-600 text-white px-6 py-3 rounded-lg text-lg font-semibold transition"
    on:click={auth}
  >
    Auth
  </button>
</main>
