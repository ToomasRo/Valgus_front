<template>
  <v-row justify="center" align="center">

    <v-spacer />
    <v-col cols="6">
      <v-combobox
        v-model="item.shelfId"
        :items="list"
        :search-input.sync="searchKey"
        auto-select-first
        clearable
        label="Shelf id"
        hide-no-data
        outlined
        @click:clear="item.shelfId = null"
      >
        <!-- hide-selected -->
        <template #append-outer>
          <v-tooltip left>
            <template #activator="{ on }">
              <v-btn
                icon
                v-on="on"
                @click="scanShelfId()"
              >
                <!-- class="pb-5 pr-2" -->
                <v-icon>{{ 'mdi-qrcode-scan' }}</v-icon>
              </v-btn>
            </template>
            <span>Scan Shelf Id</span>
          </v-tooltip>
        </template>
        <!-- <template #item="data">
          <v-list-item-content>
            <v-list-item-title>{{ data.item.select }}</v-list-item-title>
          </v-list-item-content>
        </template> -->
      </v-combobox>
      <!-- scan one shelf and add multiple items -->
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
    <v-spacer />

    <v-col cols="12">
      <v-row
        justify="end"
        dense
      >
        <v-col cols="auto">
          <v-btn outlined @click="submit()">
            Submit
          </v-btn>
        </v-col>
      </v-row>
    </v-col>

    <div ref="scannerContainer"></div>
    <video ref="video"></video>
  </v-row>
</template>

<script>
import QrScanner from 'qr-scanner';
import Quagga from 'quagga';

export default {
  name: 'IndexPage',
  data() {
    return {
      shelfIdScanner: null,
      item: {
        packageIdList: ["x", "t", "z"]
      },
      searchKey: null,
      searchingByKey: false,
      barcodeScannerIsRunning: false,
      list: ["A1-1", "A1-2", "A1-3", "A1-4", "A2-1"]
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
      console.log("Barcode detected and processed : [" + result.codeResult.code + "]", result);
      this.processBarcode(result.codeResult.code)
      this.barcodeScannerIsRunning = false
      Quagga.stop()
    });
  },

  methods: {
    scanShelfId () {
      this.shelfIdScanner.start()
    },

    processShelfId (result) {
      this.shelfIdScanner.stop()
      this.item.shelfId = result.data
    },

    scanPackageId () {
      Quagga.init({
        inputStream: {
          name: "Live",
          type: "LiveStream",
          target: this.$refs.scannerContainer,
          constraints: {
            width: 480,
            height: 320,
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
          console.log(err);
          return
        }
        console.log("Initialization finished. Ready to start");
        Quagga.start();
        this.barcodeScannerIsRunning = true;
      })

    },

    processBarcode (code) {
      if (!this.item.packageIdList.includes(code)) { 
        this.item.packageIdList.push(code)
      }
    },
    
    submit () {
      this.$axios.post('http://127.0.0.1:8000/create', this.item)
        // .then(res => {
          // const declId = this.declId || res.data
          // this.formatSaveRes(res, options)
          // this.$store.dispatch('declaration/loadHeader', { id: declId })
        // })
        // .catch(err => { this.setServerErrors(err) })
        // .finally(() => { this.$pageload.loading = false })
    }
  }

}
</script>
