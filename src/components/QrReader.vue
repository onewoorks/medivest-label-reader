<template>
    <div
        :class="{ fullscreen: fullscreen }"
        :key="componentKey"
        ref="wrapper"
        @fullscreenchange="onFullscreenChange"
    >
        <div class="qrreader">
            <center>
                <img
                    src="img/logo.png"
                    alt="nsummit logo"
                    style="height:50px"
                />
            </center>
        </div>
        <qrcode-stream @decode="onDecode" @init="onInit" />
        <a
            href="#myModal"
            role="button"
            class="btn btn-primary"
            data-backdrop="static"
            data-keyboard="false"
            data-toggle="modal"
            ref="ticketModal"
            >Launch modal</a
        >
        <div
            class="modal"
            id="myModal"
            tabindex="-1"
            role="dialog"
            aria-hidden="true"
        >
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <div class="modal-body p-2 text-left" id="result">
                        <div v-if="Object.keys(customer).length > 2">
                            <table
                                class="table table-bordered"
                                style="font-size:0.9em"
                            >
                                <tbody>
                                    <tr>
                                        <th scoped="" style="width:90px">QR</th>
                                        <td>{{ read_qr }}</td>
                                    </tr>
                                    <tr>
                                        <th scope="">Asset Name</th>
                                        <td>{{ customer.asset_info.model }}</td>
                                    </tr>
                                    <tr>
                                        <th scope="">Asset No</th>
                                        <td>
                                            {{ customer.asset_info.asset_no }}
                                        </td>
                                    </tr>
                                    <tr>
                                        <th scope="">Serial No</th>
                                        <td>
                                            {{ customer.asset_info.serial_no }}
                                        </td>
                                    </tr>
                                    <tr>
                                        <th scope="">Location</th>
                                        <td>
                                            {{
                                                customer.asset_info
                                                    .location_name
                                            }}
                                        </td>
                                    </tr>
                                    <tr
                                        v-for="(activity, index) in customer
                                            .payloads.activity"
                                        :key="index"
                                        v-show="activity.qr_code === read_qr"
                                    >
                                        <th scope="">Activity</th>
                                        <td>
                                    {{ activity.name }}
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button
                            v-if="customer.valid == 1"
                            type="button"
                            class="btn btn-warning btn-block"
                            @click="onClose"
                        >
                            {{ customer.message }}
                        </button>
                        <button
                            v-else
                            type="button"
                            @click="onConfirm"
                            class="btn btn-block btn-primary"
                            data-dismiss="modal"
                        >
                            NEXT
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import Axios from 'axios'
import { QrcodeStream } from 'vue-qrcode-reader'

export default {
    name: 'QrReader',
    components: {
        QrcodeStream
    },
    data: function() {
        return {
            customer: {},
            fullscreen: true,
            componentKey: 0,
            reader_counter: null,
            read_qr: null
        }
    },
    methods: {
        reset_reader_counter() {
            localStorage.removeItem('counter_no')
            this.reader_counter = null
        },
        onFullscreenChange(event) {
            this.fullscreen = document.fullscreenElement !== null
        },

        requestFullscreen() {
            const elem = this.$refs.wrapper

            if (elem.requestFullscreen) {
                elem.requestFullscreen()
            } else if (elem.mozRequestFullScreen) {
                /* Firefox */
                elem.mozRequestFullScreen()
            } else if (elem.webkitRequestFullscreen) {
                /* Chrome, Safari and Opera */
                elem.webkitRequestFullscreen()
            } else if (elem.msRequestFullscreen) {
                /* IE/Edge */
                elem.msRequestFullscreen()
            }
        },

        exitFullscreen() {
            if (document.exitFullscreen) {
                document.exitFullscreen()
            } else if (document.mozCancelFullScreen) {
                /* Firefox */
                document.mozCancelFullScreen()
            } else if (document.webkitExitFullscreen) {
                /* Chrome, Safari and Opera */
                document.webkitExitFullscreen()
            } else if (document.msExitFullscreen) {
                /* IE/Edge */
                document.msExitFullscreen()
            }
        },

        logErrors(promise) {
            promise.catch(console.error)
        },
        onDecode(result) {
            this.reader = result
            this.getResult(result)
        },
        onClose() {
            this.forceRerender()
        },
        onConfirm() {
            this.forceRerender()
        },
        forceRerender() {
            this.componentKey += 1
        },
        findAsset() {
            Axios.get(`${process.env.VUE_APP_URL}/data/asset_activity.json`).then(
                response => {
                    let asset = response.data.find(
                        k => k.payloads.qr_code === this.read_qr
                    )
                    this.customer = asset
                    this.$refs.ticketModal.click()
                }
            )
        },
        findAssetActivity() {
            Axios.get(`${process.env.VUE_APP_URL}/data/asset_activity.json`).then(
                response => {
                    let asset_code = this.read_qr.split('_')
                    let asset = response.data.find(
                        k => k.payloads.qr_code === asset_code[0]
                    )
                    this.customer = asset
                    this.$refs.ticketModal.click()
                }
            )
        },
        getResult(read) {
            if (read != '') {
                this.read_qr = read
                let code_len = this.read_qr.split('_')
                if (code_len.length == 1) {
                    this.findAsset()
                }
                if (code_len.length == 2) {
                    this.findAssetActivity()
                }
                // Axios.get(
                //     process.env.VUE_APP_API + `/asset/qr_code/${read}`
                // ).then(response => {
                //     var data = response.data[0]
                //     this.customer = data
                //     this.$refs.ticketModal.click()
                // })
            }
        },
        async onInit(promise) {
            try {
                await promise
            } catch (error) {
                if (error.name === 'NotAllowedError') {
                    this.error =
                        'ERROR: you need to grant camera access permisson'
                } else if (error.name === 'NotFoundError') {
                    this.error = 'ERROR: no camera on this device'
                } else if (error.name === 'NotSupportedError') {
                    this.error =
                        'ERROR: secure context required (HTTPS, localhost)'
                } else if (error.name === 'NotReadableError') {
                    this.error = 'ERROR: is the camera already in use?'
                } else if (error.name === 'OverconstrainedError') {
                    this.error = 'ERROR: installed cameras are not suitable'
                } else if (error.name === 'StreamApiNotSupportedError') {
                    this.error =
                        'ERROR: Stream API is not supported in this browser'
                }
            }
        }
    },
    mounted: function() {
        this.reader_counter = localStorage.getItem('counter_no')
        if (this.reader_counter == '') {
            localStorage.removeItem('counter_no')
        }
    }
}
</script>

<style scoped>
.btn {
    border-radius: 0;
}
.modal-dialog-centered {
    align-items: flex-end;
}

.fullscreen {
    position: fixed;
    z-index: 1000;
    top: 0;
    bottom: 0;
    right: 0;
    left: 0;
}

.fullscreen-button {
    background-color: white;
    position: absolute;
    bottom: 0;
    right: 0;
    margin: 1rem;
}
.fullscreen-button img {
    width: 2rem;
}
.qrreader {
    position: absolute;
    z-index: 1001;
    background-color: #fff;
    width: 100%;
}
.qrreader > img {
    float: left;
}
.qrreader > .detail {
    line-height: 80px;
}
</style>
