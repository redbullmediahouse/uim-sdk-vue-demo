<template>
  <div class="uim">
  <button class="uim-button" @click="handleUimButtonClick">
    <span v-if="user">{{user.name}}</span>
    <span v-else>Login</span>
  </button>
  <div v-show="showOverlay" class="overlay" ref="overlay">
    <div v-if="user">
      <img v-if="user.image" class="avatar" :src="user.image" :alt="user.name + '\'s profile picture'">
      <button class="logout-button" @click="logout">Logout</button>
    </div>
    <div class="social-logins" v-else>
      <button class="login-button" v-for="socialProvider in socialProviders" :key="socialProvider" @click="socialLogin(socialProvider)">{{socialProvider}}</button>
    </div>
  </div>
  <div v-show="notification" class="notification">New Message: {{notification && notification.msg}}</div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { IAuthenticatedUser, IUimApi, setupUimApi } from '@uim/web-sdk';

@Component
export default class UimComponent extends Vue {
    notification: {msg: string; timer: number} | null = null;
    user: {name: string, image?: string} | null = null;
    uimApi: IUimApi | null = null;
    showOverlay: boolean = false;

    get socialProviders() {
        return this.uimApi ? this.uimApi.getSocialProviders() : [];
    }

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

    async handleUimButtonClick() {
        if (!this.showOverlay && !this.user && this.uimApi) {
            try {
                this.updateUser(await this.uimApi.tryApplicationActivation());
            } catch (e) {
                console.log('application activation failed');
                this.toggleOverlay();
            }
        } else {
            this.toggleOverlay();
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
            const image = await user.getAvatar();
            this.user = { name, image };
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

.avatar {
  $diameter: 80px;
  width: $diameter;
  height: $diameter;
  border: 1px solid rgba($strokeColor, 50%);
  border-radius: $diameter / 2;
}

.notification {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  background: $backgroundColor;
}
</style>
