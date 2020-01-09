<template>
  <div class="uim">
  <button v-if="!user" class="login-button">Login</button>
  <div v-if="user" class="user">{{user.getDisplayName()}}</div>
  <div v-if="notification" class="notification">New Message: {{notification.msg}}</div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { IAuthenticatedUser, setupUimApi } from '@uim/web-sdk';

@Component
export default class UimComponent extends Vue {
    notification: {msg: string; timer: number} | null = null;
    user: IAuthenticatedUser | null = null;

    async created() {
        const uimApi = await setupUimApi()
            // IMPORTANT!!! the "design" environment is used for testing - when you go to prod remove the .withEnvironment setting!
            .withEnvironment('design')
            // IMPORTANT!!! the "design" and "production" environments will have different applicationIds
            .withApplicationId('55a61a30945dd21cec9f2001')
            .onAccountVerified(this.createNotificationHandler('account verified'))
            .onRegistrationRejected(this.createNotificationHandler('registration rejected'))
            .onNewMandatoryInformation(this.createNotificationHandler('new mandatory information'))
            .onLoginWithNoMissingItems(this.createNotificationHandler('login with no missing items'))
            .onNewPasswordRequired(this.createNotificationHandler('new password required'))
            .onEmailUpdateVerified(this.createNotificationHandler('email update verified'))
            .onAuthRedirectCallback(this.createNotificationHandler('auth redirect'))
            .onNewsletterUnsubscribed(this.createNotificationHandler('newsletter unsubscribed'))
            .onTokenRefreshed(this.createNotificationHandler('token refreshed'))
            .onSessionInvalid(this.createNotificationHandler('session invalid'))
            .onWorkflowError(this.createNotificationHandler('workflow error'))
            .asStaging(false)
            .withCountry('AT')
            .withLanguage('de')
            .withAutoRefresh(false)
            .initialize();
        try {
            this.user = await uimApi.getAuthenticatedUser();
        } catch (e) {
            console.log('no user logged in');
        }
    }

    createNotificationHandler(msg: string): VoidFunction {
        return () => this.showNotification(msg);
    }

    showNotification(msg: string): void {
        if (this.notification) {
            clearTimeout(this.notification.timer);
        }
        const timer = setTimeout(() => {
            this.notification = null;
        }, 5000);
        this.notification = { msg, timer };
    }
}

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
button,
button:hover,
button:focus,
button:active,
button:disabled,
button:disabled:hover {
  padding: 0;
  border: none;
  outline: none;
  font: inherit;
  color: inherit;
  background: none;
  cursor: pointer;
  -webkit-appearance: none;
  -moz-appearance: none;
  font-weight: bold;
}

button.login-button {
  color: #2c3e50;
  text-decoration: underline;
}

.popup {
  position: fixed;
  bottom: 0;
  width: 100%;
  background: honeydew;
}
</style>
