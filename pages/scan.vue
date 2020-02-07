<template>
    <v-container fluid>
        <v-card>
            <v-row px-3>
                <v-spacer></v-spacer>
                <v-col xs="12" md="4">
                    <v-text-field
                        v-model="barcode"
                        @focus="showScanner = true"
                        @click="showScanner = true"
                        @blur="showScanner = false"
                        label="Barcode"
                        append-icon="barcode-scanner"
                        readonly
                    ></v-text-field>

                    <v-text-field
                        v-model="description"
                        label="Description"
                        readonly
                    ></v-text-field>

                    <v-text-field
                        v-model="value"
                        label="Value"
                        type="number"
                    ></v-text-field>
                </v-col>
                <v-spacer></v-spacer>
            </v-row>

            <v-row>
                <v-spacer></v-spacer>
                <v-col xs="12" md="4" class="pb-3">
                    <v-btn @click="save" color="success" class="mr-3">
                        Save
                    </v-btn>
                    <v-btn @click="showScanner = !showScanner" color="primary">
                        Scan
                    </v-btn>
                </v-col>
                <v-spacer></v-spacer>
            </v-row>

            <v-row v-show="showScanner" mt-3>
                <v-col xs="12">
                    <div id="scandit" style="max-height:450px"></div>
                </v-col>
            </v-row>
        </v-card>
    </v-container>
</template>

<script lang="ts">
import Vue from 'vue'
import { Barcode, BarcodePicker, ScanSettings, configure } from 'scandit-sdk'

export default Vue.extend({
    name: 'SCFP',
    data() {
        return {
            barcode: null,
            description: '',
            value: null,
            showScanner: false,
            scanInterval: 1000,
            barcodePicker: {} as BarcodePicker
        }
    },
    computed: {
        scannerWidth() {
            let width = window.innerWidth

            const el = document.querySelector('.scandit-barcode-picker')

            if (el != null) {
                width = el.clientWidth
            }

            return width
        },
        isInstalledApp() {
            if (window.matchMedia('(display-mode: standalone)').matches) {
                return true
            } else {
                return false
            }
        }
    },
    watch: {
        showScanner() {
            if (this.showScanner && this.barcodePicker.isScanningPaused) {
                this.barcodePicker.resumeScanning()
            } else if (!this.showScanner) {
                this.barcodePicker.pauseScanning()
            }
        },
        barcode() {
            this.getDescription()
        }
    },
    beforeMount() {
        if (this.isInstalledApp || window.location.hostname === 'localhost') {
            this.initScandit()
        }
    },
    mounted() {
        if (this.isInstalledApp || window.location.hostname === 'localhost') {
            this.createScandit()
        }
    },
    methods: {
        initScandit() {
            configure(process.env.VUE_APP_SCANDIT_LICENSE as string, {
                engineLocation: 'https://cdn.jsdelivr.net/npm/scandit-sdk/build'
            })
        },
        createScandit() {
            const el = document.getElementById('scandit')

            if (el != null) {
                BarcodePicker.create(el, {
                    playSoundOnScan: true,
                    vibrateOnScan: true,
                    scanningPaused: true,
                    videoFit: BarcodePicker.ObjectFit.COVER
                }).then((barcodePicker) => {
                    this.barcodePicker = barcodePicker

                    const scanSettings = new ScanSettings({
                        enabledSymbologies: [Barcode.Symbology.CODE128],
                        codeDuplicateFilter: this.scanInterval,
                        searchArea: {
                            x: 0,
                            y: 0.35,
                            height: 0.15,
                            width: 1
                        }
                    })

                    barcodePicker.applyScanSettings(scanSettings)
                    barcodePicker.setMirrorImageEnabled(false)

                    barcodePicker.on('scan', (scanResult) => {
                        this.barcode = scanResult.barcodes[0].data
                    })
                })
            }
        },
        async getDescription() {
            try {

            } catch (err) {
                console.log(err.response)
            }
        },
        save() {}
    }
})
</script>
