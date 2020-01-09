<template>
  <div class="uim">
  <button class="uim-button" @click="toggleOverlay">
    <span v-if="!user">Login</span>
    <span v-if="user">{{user.name}}</span>
  </button>
  <div v-if="showOverlay" class="overlay" ref="overlay">
    <div class="social-logins" v-if="!user">
      <button class="login-button" v-for="socialProvider in uimApi.getSocialProviders()" :key="socialProvider" @click="socialLogin(socialProvider)">{{socialProvider}}</button>
    </div>
    <button v-if="user" class="logout-button" @click="logout">Logout</button>
  </div>
  <div v-if="notification" class="notification">New Message: {{notification.msg}}</div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { IAuthenticatedUser, IUimApi, setupUimApi } from '@uim/web-sdk';

@Component
export default class UimComponent extends Vue {
    notification: {msg: string; timer: number} | null = null;
    user: {name: string} | null = null;
    uimApi?: IUimApi;
    showOverlay: boolean = false;

    async created() {
        this.uimApi = await setupUimApi()
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
            this.updateUser(await this.uimApi.getAuthenticatedUser());
        } catch (e) {
            console.log('no user logged in');
        }
    }

    toggleOverlay() {
        this.showOverlay = !this.showOverlay;
    }

    async socialLogin(socialMediaName: string) {
        if (this.uimApi) {
            this.updateUser(await this.uimApi.socialLogin(socialMediaName));
            this.showOverlay = false;
        }
    }

    async logout() {
        if (this.uimApi) {
            await this.uimApi.logout();
            await this.updateUser(null);
            this.showOverlay = false;
        }
    }

    async updateUser(user: IAuthenticatedUser | null) {
        if (user) {
            const name = await user.getDisplayName();
            this.user = { name };
        } else {
            this.user = null;
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

$backgroundColor: #ecf8f3;
$strokeColor: #2c3e50;

.uim {
  display: inline;
  position: relative;
}

button.uim-button {
  color: $strokeColor;
  text-decoration: underline;
}

.overlay {
  position: absolute;
  top: 1.5em;
  right: 0;
  border: 1px solid grey;
  background: $backgroundColor;
  padding: 0.5rem;
}

.social-logins {
  display: flex;
  flex-direction: column;
  align-content: stretch;
}

button.login-button,
button.logout-button {
  background: white;
  margin-bottom: 0.25rem;
  padding: 0.25rem;
  border: 1px solid $strokeColor;
}

.notification {
  position: fixed;
  top: 0;
  width: 100%;
  background: $backgroundColor;
}
</style>
