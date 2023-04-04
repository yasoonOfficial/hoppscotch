<template>
  <AppShortcuts :show="showShortcuts" @close="showShortcuts = false" />
  <AppShare :show="showShare" @hide-modal="showShare = false" />
  <FirebaseLogin :show="showLogin" @hide-modal="showLogin = false" />
</template>

<script setup lang="ts">
import { cons } from "fp-ts/lib/ReadonlyNonEmptyArray";
import { ref } from "vue";
import { defineActionHandler } from "~/helpers/actions";

const showShortcuts = ref(false);
const showShare = ref(false);
const showLogin = ref(false);

defineActionHandler("flyouts.keybinds.toggle", () => {
  showShortcuts.value = !showShortcuts.value;
});

defineActionHandler("modals.share.toggle", () => {
  showShare.value = !showShare.value;
});

const msalConfig = {
  auth: {
    clientId: "d5953f6b-d93f-4d03-a95d-d39e9c96d842",
    authority: "https://login.microsoftonline.com/0e7219f7-4c4d-47e7-b98a-7b9b9de02b80",
    redirectUri: "https://hop.yasoon.org",
  },
  cache: {
    cacheLocation: "localStorage", // This configures where your cache will be stored
    storeAuthStateInCookie: false, // Set this to "true" if you are having issues on IE11 or Edge
  },
};

//@ts-ignore
const myMSALObj = new msal.PublicClientApplication(msalConfig);
const apiConfig = {
  uri: "https://hop.yasoon.org/proxy", // e.g. http://localhost:5000/api
  scopes: ["api://d5953f6b-d93f-4d03-a95d-d39e9c96d842/Access.CustomerInstance"] // e.g. [api//middle_tier_api_client_id/.default]
};
//@ts-ignore
const loginRequest = {
  scopes: ["openid", "profile"]
};
const tokenRequest: any = {
  scopes: [...apiConfig.scopes],
};

defineActionHandler("modals.login.toggle", async () => {
  const currentAccounts = myMSALObj.getAllAccounts();
  let userName = "";
  if (!currentAccounts || currentAccounts.length < 1) {
    let loginResponse = await myMSALObj.loginPopup(loginRequest);
    userName = loginResponse.account.username;
  }
  else {
    userName = currentAccounts[0].username;
  }

  tokenRequest.account = myMSALObj.getAccountByUsername(userName);
  let tokenResponse = await myMSALObj.acquireTokenPopup(tokenRequest);
  window["PROXY_ACCESS_TOKEN"] = tokenResponse.accessToken;
  /*
  myMSALObj.acquireTokenSilent(tokenRequest)
    .catch((error: any) => {
      console.warn(error);
      console.warn("silent token acquisition fails. acquiring token using popup");
      //@ts-ignore
      if (error instanceof msal.InteractionRequiredAuthError) {
        // fallback to interaction when silent call fails
        return 
      } else {
        console.warn(error);
      }
    });
*/
})

</script>
