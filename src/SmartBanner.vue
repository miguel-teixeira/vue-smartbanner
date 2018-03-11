<template>
    <div class="smart-banner">
        <transition appear :name="transition">
            <smart-banner-panel
                v-if="showPanel"
                @close="close()"
                @open="open()"
                :app-icon="appIcon"
                :app-title="appTitle"
                :app-company="appCompany"
                :app-availability="appAvailability"
                :app-url="appUrl"
                :open-text="openText"
            />
            </transition>
    </div>
</template>

<script>
    import SmartBannerPanel from './SmartBannerPanel';
    import UAParser from 'ua-parser-js';
    import cookies from 'js-cookie';

    const agent = new UAParser();

    export default {
        name: "smart-banner",
        data() {
            return {
                show: false
            }
        },
        computed: {
            isMobilePhone() {
                return this.isPhone && (this.isIOS || this.isAndroid);
            },
            isPhone() {
                return agent.getDevice().type === 'mobile';
            },
            isIOS() {
                return agent.getOS().name === 'iOS';
            },
            isAndroid() {
                return agent.getOS().name === 'Android';
            },
            hasClosedBanner() {
                return cookies.get('hasClosedMobileAppBanner') === "true";
            },
            hasOpenedStorePage() {
                return cookies.get('hasOpenedMobileAppStore') === "true";
            },
            showPanel() {
                return this.show
                    && ! this.hasClosedBanner
                    && ! this.hasOpenedStorePage
                    && this.isMobilePhone;
            },
            appAvailability() {
                return this.isAndroid
                    ? `${this.googlePlayPrice} - ${this.googlePlayDescription}`
                    : `${this.appStorePrice} - ${this.appStoreDescription}`;
            },
            appUrl() {
                return this.isAndroid
                    ? this.googlePlayUrl
                    : this.appStoreUrl;
            }
        },
        methods: {
            daysInSeconds(days) {
                return days * 24 * 60 * 60;
            },
            close() {
                cookies.set(
                    'hasClosedMobileAppBanner',
                    true,
                    {expires: this.daysInSeconds(this.daysHidden)}
                );

                this.show = false;
            },
            open() {
                cookies.set(
                    'hasOpenedMobileAppStore',
                    true,
                    {expires: this.daysInSeconds(this.daysReminder)}
                );
                this.show = false;
            }
        },
        components: {
            SmartBannerPanel,
        },
        props: {
            transition: {
                default: 'slideY'
            },
            appearDelay: {
                default: 1000
            },
            daysHidden: {
                default: 15
            },
            daysReminder: {
                default: 90
            },
            appIcon: {
                default: ''
            },
            appTitle: {
                default: 'App Title'
            },
            appCompany: {
                default: 'Company'
            },
            googlePlayPrice: {
                default: 'FREE'
            },
            appStorePrice: {
                default: 'FREE'
            },
            googlePlayDescription: {
                default: 'Google Play'
            },
            appStoreDescription: {
                default: 'App Store'
            },
            googlePlayUrl: {
                default: 'https://play.google.com'
            },
            appStoreUrl: {
                default: 'https://www.apple.com/itunes'
            },
            openText: {
                default: 'Open'
            }
        },
        mounted() {
            setTimeout(() => this.show = true, this.appearDelay);
        }
    }
</script>

<style lang="scss">
    .smart-banner {

        @keyframes slideDown {
            from {
                transform: translateY(-100%);
            }
            to {
                transform: translateY(0);
            }
        }

        @keyframes slideUp {
            from {
                transform: translateY(0);
            }
            to {
                transform: translateY(-100%);
            }
        }

        .slideY-enter {
            height: 0;
        }

        .slideY-enter-active {
            animation: slideDown .3s ease-in;
            transition: height .3s ease-in;
        }

        .slideY-leave-active {
            transition: height .3s ease-out;
            animation: slideUp .3s ease-out;
            height: 0;
        }
    }
</style>

