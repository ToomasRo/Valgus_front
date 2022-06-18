<template>
  <v-row justify="center" align="center">
    <v-col cols="12">
      <v-autocomplete
        v-model="item.shelfId"
        :items="list"
        :search-input.sync="shelfSearch"
        auto-select-first
        clearable
        label="Shelf id"
        outlined
        @click:clear="item.shelfId = null"
      >
        <template #no-data>
          <v-list-item>
            <v-list-item-content>
              <v-list-item-title>
                Shelf "<strong>{{ shelfSearch }}</strong>" does not exist
              </v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </template>
        <template #append-outer>
          <v-tooltip left>
            <template #activator="{ on }">
              <v-btn
                icon
                v-on="on"
                @click="scanShelfId()"
              >
                <v-icon>{{ 'mdi-qrcode-scan' }}</v-icon>
              </v-btn>
            </template>
            <span>Scan Shelf Id</span>
          </v-tooltip>
        </template>
      </v-autocomplete>
      <v-combobox
        v-model="item.packageIdList"
        :items="[]"
        :maxlength="13"
        auto-select-first
        clearable
        label="Package ids"
        hide-no-data
        outlined
        multiple
        chips
        @click:clear="item.packageIdList = null"
      >
        <template #append-outer>
          <v-tooltip left>
            <template #activator="{ on }">
              <v-btn
                icon
                v-on="on"
                @click="scanPackageId()"
              >
                <v-icon>{{ 'mdi-barcode-scan' }}</v-icon>
              </v-btn>
            </template>
            <span>Scan Package Id</span>
          </v-tooltip>
        </template>
      </v-combobox>
    </v-col>

    <v-col cols="12">
      <v-row
        justify="end"
      >
        <v-col cols="auto">
          <v-btn outlined @click="submit()">
            Submit
          </v-btn>
        </v-col>
      </v-row>
    </v-col>

    <v-snackbar v-model="successSnackbar">
      <v-row>
        <v-col cols="auto">
          Package has been saved
        </v-col>
        <v-spacer></v-spacer>
        <v-col cols="auto">
          <v-btn icon @click.native="successSnackbar = false">
            <v-icon>{{ 'mdi-close' }}</v-icon>
          </v-btn>
        </v-col>
      </v-row>
    </v-snackbar>

    <v-dialog
      v-model="shelfScannerDialog"
      eager
      fullscreen
      hide-overlay
      transition="dialog-bottom-transition"
    >
      <v-card>
        <video
          ref="video"
          width="100%"
          height="100%"
        ></video>
        <v-row justify="end">
          <v-col cols="auto">
            <v-btn outlined @click="shelfScannerDialog = false">
              Close
            </v-btn>
          </v-col>
        </v-row>
      </v-card>
    </v-dialog>

    <v-dialog
      v-model="packageScannerDialog"
      eager
      fullscreen
      hide-overlay
      transition="dialog-bottom-transition"
    >
      <v-card>
        <div ref="scannerContainer"></div>
        {{ item.packageIdList }}
        <v-row justify="end">
          <v-col cols="auto">
            <v-btn outlined @click="packageScannerDialog = false">
              Close
            </v-btn>
          </v-col>
        </v-row>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<script>
import QrScanner from 'qr-scanner';
import Quagga from 'quagga';

export default {
  name: 'IndexPage',
  data() {
    return {
      successSnackbar: false,
      shelfIdScanner: null,
      item: {
        shelfId: null,
        packageIdList: []
      },
      shelfSearch: null,
      searchingByKey: false,
      shelfScannerDialog: false,
      packageScannerDialog: false,
      barcodeScannerIsRunning: false,
      list: [
        "A1-1", "A1-2", "A1-3", "A1-4",
        "A2-1", "A2-2", "A2-3", "A2-4",
        "B1-1", "B1-2", "B1-3", "B1-4",
        "B2-1", "B2-2", "B2-3", "B2-4",
        ]
    }
  },

  computed: {
    shelfIdScannerIsActive () {
      return this.shelfIdScanner?._active
    },
  },

  watch: {
    shelfScannerDialog (value) {
      if (!value) this.cancelShelfScanning()
    },

    packageScannerDialog (value) {
      if (!value) this.cancelPackageScanning()
    }
  },

  mounted() {
    this.shelfIdScanner = new QrScanner(this.$refs.video, result => this.processShelfId(result), {
      highlightScanRegion: true
    })

    Quagga.onProcessed(function (result) {
    const drawingCtx = Quagga.canvas.ctx.overlay
    const drawingCanvas = Quagga.canvas.dom.overlay

    if (result) {
      if (result.boxes) {
        drawingCtx.clearRect(0, 0, parseInt(drawingCanvas.getAttribute("width")), parseInt(drawingCanvas.getAttribute("height")));
        result.boxes.filter(function (box) {
            return box !== result.box;
        }).forEach(function (box) {
            Quagga.ImageDebug.drawPath(box, { x: 0, y: 1 }, drawingCtx, { color: "green", lineWidth: 2 });
        });
      }

      if (result.box) {
        Quagga.ImageDebug.drawPath(result.box, { x: 0, y: 1 }, drawingCtx, { color: "#00F", lineWidth: 2 });
      }

      if (result.codeResult && result.codeResult.code) {
        Quagga.ImageDebug.drawPath(result.line, { x: 'x', y: 'y' }, drawingCtx, { color: 'red', lineWidth: 3 });
      }
    }
    });

    Quagga.onDetected((result) => {
      this.processBarcode(result.codeResult.code)
    });

    // this.scanPackageId()
    // this.scanShelfId()
  },

  methods: {
    scanShelfId () {
      this.shelfScannerDialog = true
      this.shelfIdScanner.start()
    },

    processShelfId (result) {
      this.shelfScannerDialog = false
      this.item.shelfId = result.data
    },

    processBarcode (code) {
      const codeIsNotRegistered = !this.item.packageIdList.includes(code)
      const correctFormat = /[A-z]{2}[0-9]{9}[A-z]{2}/
      const codeFormatIsCorrect = code.match(correctFormat) 
      if (codeIsNotRegistered && code.length === 13 && codeFormatIsCorrect) {
        this.item.packageIdList.push(code)
      }
    },

    submit () {
      this.$axios.post('http://127.0.0.1:8000/create', this.item)
        .then(({ status, data }) => {
          if (status === 200) {
            this.successSnackbar = true
            this.item = {}
          }
        })
        // .catch(err => { this.setServerErrors(err) })
        // .finally(() => { this.$pageload.loading = false })
    },

    cancelShelfScanning () {
      this.shelfIdScanner.stop()
    },

    cancelPackageScanning () {
        this.barcodeScannerIsRunning = false
        Quagga.stop()
    },

    scanPackageId () {
      Quagga.init({
        inputStream: {
          name: "Live",
          type: "LiveStream",
          target: this.$refs.scannerContainer,
          constraints: {
            // width: 480,
            // height: 320,
            facingMode: "environment"
          }
        },
        decoder: {
          readers: [
              "code_128_reader",
          ],
          debug: {
              showCanvas: true,
              showPatches: true,
              showFoundPatches: true,
              showSkeleton: true,
              showLabels: true,
              showPatchLabels: true,
              showRemainingPatchLabels: true,
              boxFromPatches: {
                  showTransformed: true,
                  showTransformedBox: true,
                  showBB: true
              }
          }
        }
      }, (err) => {
        if (err) {
          console.log(err)
          return
        }
        this.barcodeScannerIsRunning = true
        this.packageScannerDialog = true
        Quagga.start()
      })

    },
  }

}
</script>

<style>
  .drawingBuffer {
    top: 0px !important;
    left: 0px !important;
    position: absolute;
  }
</style>
