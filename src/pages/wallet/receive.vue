<template>
<q-page class="receive">
    <q-list link no-border :dark="theme=='dark'" class="kredits-list">

        <q-list-header>{{ $t("strings.addresses.myPrimaryAddress") }}</q-list-header>
        <receive-item
            class="primary-address"
            v-for="address in address_list.primary"
            :key="address.address"
            :address="address"
            :sublabel="$t('strings.addresses.primaryAddress')"
            :showQR="showQR"
            :copyAddress="copyAddress"
            :details="details"
            whiteQRIcon
        />

        <template v-if="address_list.used.length">
            <q-list-header>{{ $t("strings.addresses.myUsedAddresses") }}</q-list-header>
            <receive-item
                v-for="address in address_list.used"
                :key="address.address"
                :address="address"
                :sublabel="`${$t('strings.addresses.subAddress')} (${$t('strings.addresses.subAddressIndex', { index: address.address_index })})`"
                :showQR="showQR"
                :copyAddress="copyAddress"
                :details="details"
            />
        </template>

        <template v-if="address_list.unused.length">
            <q-list-header>{{ $t("strings.addresses.myUnusedAddresses") }}</q-list-header>
            <receive-item
                v-for="address in address_list.unused"
                :key="address.address"
                :address="address"
                :sublabel="`${$t('strings.addresses.subAddress')} (${$t('strings.addresses.subAddressIndex', { index: address.address_index })})`"
                :showQR="showQR"
                :copyAddress="copyAddress"
                :details="details"
                :shouldShowInfo="false"
            />
        </template>

    </q-list>
    <AddressDetails ref="addressDetails" />

    <!-- QR Code -->
    <template v-if="QR.address != null">
        <q-modal v-model="QR.visible" minimized :content-css="{padding: '25px'}">

            <div class="text-center q-mb-sm q-pa-md" style="background: white;">
                <qrcode-vue :value="QR.address" size="240" ref="qr">
                </qrcode-vue>
                <q-context-menu>
                    <q-list link separator style="min-width: 150px; max-height: 300px;">
                        <q-item v-close-overlay @click.native="copyQR()">
                            <q-item-main :label="$t('menuItems.copyQR')" />
                        </q-item>
                        <q-item v-close-overlay @click.native="saveQR()">
                            <q-item-main :label="$t('menuItems.saveQR')" />
                        </q-item>
                    </q-list>
                </q-context-menu>
            </div>

            <q-btn
                 color="primary"
                 @click="QR.visible = false"
                 :label="$t('buttons.close')"
             />
        </q-modal>
    </template>
</q-page>
</template>

<script>

const { clipboard, nativeImage } = require("electron")
import { mapState } from "vuex"
import QrcodeVue from "qrcode.vue";
import AddressDetails from "components/address_details"
import ReceiveItem from "components/receive_item"

export default {
    computed: mapState({
        theme: state => state.gateway.app.config.appearance.theme,
        address_list: state => state.gateway.wallet.address_list
    }),
    filters: {
        toString: function (value) {
            if (typeof value !== "number") return "N/A";
            return String(value);
        },
        currency: function (value) {
            if (typeof value !== "number") return "N/A";

            const amount = value / 1e8
            return amount.toLocaleString()
        }
    },
    data () {
        return {
            QR: {
                visible: false,
                address: null,
            }
        }
    },
    methods: {
        details (address) {
            console.log(address)
            this.$refs.addressDetails.address = address;
            this.$refs.addressDetails.isVisible = true;
        },
        showQR (address, event) {
            event.stopPropagation()
            this.QR.visible = true
            this.QR.address = address
        },
        copyQR () {
            const data = this.$refs.qr.$el.childNodes[0].toDataURL()
            const img = nativeImage.createFromDataURL(data)
            clipboard.writeImage(img)
             this.$q.notify({
                type: "positive",
                timeout: 1000,
                message: this.$t("notification.positive.qrCopied")
            })
        },
        saveQR () {
            let img = this.$refs.qr.$el.childNodes[0].toDataURL()
            this.$gateway.send("core", "save_png", {img, type: "QR Code"})
        },
        copyAddress (address, event) {
            event.stopPropagation()
            for(let i = 0; i < event.path.length; i++) {
                if(event.path[i].tagName == "BUTTON") {
                    event.path[i].blur()
                    break
                }
            }
            clipboard.writeText(address)
            this.$q.notify({
                type: "positive",
                timeout: 1000,
                message: this.$t("notification.positive.addressCopied")
            })
        }
    },
    components: {
        AddressDetails,
        QrcodeVue,
        ReceiveItem
    }
}
</script>

<style lang="scss">
.receive {
    .q-item-label {
        font-weight: 400;
    }

    .q-item-sublabel, .q-list-header {
        font-size: 13px;
    }

    .kredits-list-item {
        cursor: pointer;

        .q-item {
            padding-top: 4px;
            padding-bottom: 4px;
        }

        .q-item-side {
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .info {
            span {
                font-size: 14px;
            }

             .value {
                font-size: 16px;
                font-weight: bold;
            }
        }
    }
}
</style>
