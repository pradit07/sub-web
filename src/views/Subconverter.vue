<template>
  <div>
    <el-row style="margin-top: 10px">
      <el-col>
        <el-card>
          <div slot="header">
            Subscription Converter
             </div>
          <el-container>
            <el-form :model="form" label-width="80px" label-position="left" style="width: 100%">
              <el-form-item label="Mode:">
                <el-radio v-model="advanced" label="1">Mode Standar</el-radio>
                <el-radio v-model="advanced" label="2">Mode Lanjutan</el-radio>
              </el-form-item>
              <el-form-item label="Isi Config:">
                <el-input
                  v-model="form.sourceSubUrl"
                  type="textarea"
                  rows="3"
                  placeholder="Mendukung tautan berlangganan atau ss/ssr/vmess, beberapa tautan satu per baris atau dipisahkan oleh |"
                  @blur="saveSubUrl"
                />
              </el-form-item>
              <el-form-item label="Klien:">
                <el-select v-model="form.clientType" style="width: 100%">
                  <el-option v-for="(v, k) in options.clientTypes" :key="k" :label="k" :value="v"></el-option>
                </el-select>
              </el-form-item>

              <div v-if="advanced === '2'">
                <el-form-item label="Alamat:">
                  <el-autocomplete
                    style="width: 100%"
                    v-model="form.customBackend"
                    :fetch-suggestions="backendSearch"
                    placeholder="contoh：http://127.0.0.1:25500/sub?"
                  >
                    <el-button slot="append" @click="gotoGayhub" icon="el-icon-link">Buka repositori proyek</el-button>
                  </el-autocomplete>
                </el-form-item>
                <el-form-item label="Konfigurasi jarak jauh:">
                  <el-select
                    v-model="form.remoteConfig"
                    allow-create
                    filterable
                    placeholder="tolong pilih"
                    style="width: 100%"
                  >
                    <el-option-group
                      v-for="group in options.remoteConfig"
                      :key="group.label"
                      :label="group.label"
                    >
                      <el-option
                        v-for="item in group.options"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value"
                      ></el-option>
                    </el-option-group>
                    <el-button slot="append" @click="gotoRemoteConfig" icon="el-icon-link">Contoh konfigurasi</el-button>
                  </el-select>
                </el-form-item>
                <el-form-item label="Include:">
                  <el-input v-model="form.includeRemarks" placeholder="Kata kunci yang terkandung dalam nama simpul, mendukung reguler" />
                </el-form-item>
                <el-form-item label="Exclude:">
                  <el-input v-model="form.excludeRemarks" placeholder="Kata kunci tidak termasuk dalam nama simpul, dukungan reguler" />
                </el-form-item>
                <el-form-item label="FileName:">
                  <el-input v-model="form.filename" placeholder="Nama file langganan yang dikembalikan" />
                </el-form-item>
                <el-form-item label-width="0px">
                  <el-row type="flex">
                    <el-col>
                      <el-checkbox v-model="form.nodeList" label="Hanya Proxy" border></el-checkbox>
                    </el-col>
                    <el-popover placement="bottom" v-model="form.extraset">
                      <el-row>
                        <el-checkbox v-model="form.emoji" label="Emoji"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.scv" label="lewati verifikasi sertifikat"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.udp" @change="needUdp = true" label="Memungkinkan UDP"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.appendType" label="Jenis simpul"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.sort" label="Urutkan simpul"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.fdn" label="Saring node ilegal"></el-checkbox>
                      </el-row>
                      <el-button slot="reference">lebih banyak pilihan</el-button>
                    </el-popover>
                    <el-popover placement="bottom" style="margin-left: 10px">
                      <el-row>
                        <el-checkbox v-model="form.tpl.surge.doh" label="Surge.DoH"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.tpl.clash.doh" label="Clash.DoH"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.insert" label="NetEase Cloud"></el-checkbox>
                      </el-row>
                      <el-button slot="reference">fungsi kustom</el-button>
                    </el-popover>
                  </el-row>
                </el-form-item>
              </div>

              <div style="margin-top: 50px"></div>

              <el-divider content-position="center">
                <i class="el-icon-magic-stick"></i>
              </el-divider>

              <el-form-item label="langganan khusus:">
                <el-input class="copy-content" disabled v-model="customSubUrl">
                  <el-button
                    slot="append"
                    v-clipboard:copy="customSubUrl"
                    v-clipboard:success="onCopy"
                    ref="copy-btn"
                    icon="el-icon-document-copy"
                  >salinan</el-button>
                </el-input>
              </el-form-item>
              <el-form-item label="link pendek:">
                <el-input class="copy-content" disabled v-model="curtomShortSubUrl">
                  <el-button
                    slot="append"
                    v-clipboard:copy="curtomShortSubUrl"
                    v-clipboard:success="onCopy"
                    ref="copy-btn"
                    icon="el-icon-document-copy"
                  >salinan</el-button>
                </el-input>
              </el-form-item>

              <el-form-item label-width="0px" style="margin-top: 40px; text-align: center">
                <el-button
                  style="width: 120px"
                  type="danger"
                  @click="makeUrl"
                  :disabled="form.sourceSubUrl.length === 0"
                >Gasken</el-button>
                <el-button
                  style="width: 120px"
                  type="danger"
                  @click="makeShortUrl"
                  :loading="loading"
                  :disabled="customSubUrl.length === 0"
                >Tautan pendek</el-button>
                <!-- <el-button style="width: 120px" type="primary" @click="surgeInstall" icon="el-icon-connection">Impor sekali klik Surge</el-button> -->
              </el-form-item>

              <el-form-item label-width="0px" style="text-align: center">
                <el-button
                  style="width: 120px"
                  type="primary"
                  @click="dialogUploadConfigVisible = true"
                  icon="el-icon-upload"
                  :loading="loading"
                >Unggah</el-button>
                <el-button
                  style="width: 120px"
                  type="primary"
                  @click="clashInstall"
                  icon="el-icon-connection"
                  :disabled="customSubUrl.length === 0"
                >Impor Ke Clash</el-button>
              </el-form-item>
            </el-form>
          </el-container>
        </el-card>
      </el-col>
    </el-row>

    <el-dialog
      :visible.sync="dialogUploadConfigVisible"
      :show-close="false"
      :close-on-click-modal="false"
      :close-on-press-escape="false"
      width="700px"
    >
      <div slot="title">
        Remote config upload
        <el-popover trigger="hover" placement="right" style="margin-left: 10px">
          <el-link type="primary" :href="sampleConfig" target="_blank" icon="el-icon-info">Konfigurasi referensi</el-link>
          <i class="el-icon-question" slot="reference"></i>
        </el-popover>
      </div>
      <el-form label-position="left">
        <el-form-item prop="uploadConfig">
          <el-input
            v-model="uploadConfig"
            type="textarea"
            :autosize="{ minRows: 15, maxRows: 15}"
            maxlength="5000"
            show-word-limit
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="uploadConfig = ''; dialogUploadConfigVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="confirmUploadConfig"
          :disabled="uploadConfig.length === 0"
        >Tentu</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
const project = process.env.VUE_APP_PROJECT
const remoteConfigSample = process.env.VUE_APP_SUBCONVERTER_REMOTE_CONFIG
const gayhubRelease = process.env.VUE_APP_BACKEND_RELEASE
const defaultBackend = process.env.VUE_APP_SUBCONVERTER_DEFAULT_BACKEND + '/sub?'
const shortUrlBackend = process.env.VUE_APP_MYURLS_DEFAULT_BACKEND + '/short'
const configUploadBackend = process.env.VUE_APP_CONFIG_UPLOAD_BACKEND + '/config/upload'
const tgBotLink = process.env.VUE_APP_BOT_LINK

export default {
  data() {
    return {
      backendVersion: "",
      advanced: "2",

      isPC: true,

      options: {
        clientTypes: {
          Clash: "clash",
          Surge3: "surge&ver=3",
          Surge4: "surge&ver=4",
          Quantumult: "quan",
          QuantumultX: "quanx",
          Surfboard: "surfboard",
          Loon: "loon",
          SSAndroid: "sssub",
          V2Ray: "v2ray",
          ss: "ss",
          ssr: "ssr",
          ssd: "ssd",
          ClashR: "clashr",
          Surge2: "surge&ver=2",
        },
        backendOptions: [{ value: "http://127.0.0.1:25500/sub?" }],
        remoteConfig: [
          {
            label: "universal",
            options: [
              {
                label: "No-Urltest",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/universal/no-urltest.ini"
              },
              {
                label: "Urltest",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/universal/urltest.ini"
              }
            ]
          },
          {
            label: "customized",
            options: [
              {
                label: "Maying",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/maying.ini"
              },
              {
                label: "Ytoo",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/ytoo.ini"
              },
              {
                label: "FlowerCloud",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/flowercloud.ini"
              },
              {
                label: "Nexitally",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/nexitally.ini"
              },
              {
                label: "SoCloud",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/socloud.ini"
              },
              {
                label: "ARK",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/ark.ini"
              },
              {
                label: "ssrCloud",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/ssrcloud.ini"
              }
            ]
          },
          {
            label: "Special",
            options: [
              {
                label: "NeteaseUnblock(hanya aturan，No-Urltest)",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/special/netease.ini"
              },
              {
                label: "Basic(仅GEOIP CN + Final)",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/special/basic.ini"
              }
            ]
          }
        ]
      },
      form: {
        sourceSubUrl: "",
        clientType: "",
        customBackend: "",
        remoteConfig: "",
        excludeRemarks: "",
        includeRemarks: "",
        filename: "",
        emoji: true,
        nodeList: false,
        extraset: false,
        sort: false,
        udp: false,
        tfo: false,
        scv: false,
        fdn: false,
        appendType: false,
        insert: false, // 是否插入默认订阅的节点，对应配置项 insert_url
        new_name: true, // 是否使用 Clash 新字段

        // tpl 定制功能
        tpl: {
          surge: {
            doh: false // dns 查询是否使用 DoH
          },
          clash: {
            doh: false
          }
        }
      },

      loading: false,
      customSubUrl: "",
      curtomShortSubUrl: "",

      dialogUploadConfigVisible: false,
      uploadConfig: "",
      uploadPassword: "",
      myBot: tgBotLink,
      sampleConfig: remoteConfigSample,

      needUdp: false, // 是否需要添加 udp 参数
    };
  },
  created() {
    document.title = "Subscription Converter";
    this.isPC = this.$getOS().isPc;

    // 获取 url cache
    if (process.env.VUE_APP_USE_STORAGE === 'true') {
      this.form.sourceSubUrl = this.getLocalStorageItem('sourceSubUrl')
    }
  },
  mounted() {
    this.form.clientType = "clash";
    this.notify();
    this.getBackendVersion();
  },
  methods: {
    onCopy() {
      this.$message.success("Copied!");
    },
    goToProject() {
      window.open(project);
    },
    gotoGayhub() {
      window.open(gayhubRelease);
    },
    gotoRemoteConfig() {
      window.open(remoteConfigSample);
    },
    clashInstall() {
      if (this.customSubUrl === "") {
        this.$message.error("Silakan isi bidang yang diperlukan terlebih dahulu untuk menghasilkan tautan berlangganan");
        return false;
      }

      const url = "clash://install-config?url=";
      window.open(
        url +
          encodeURIComponent(
            this.curtomShortSubUrl !== ""
              ? this.curtomShortSubUrl
              : this.customSubUrl
          )
      );
    },
    surgeInstall() {
      if (this.customSubUrl === "") {
        this.$message.error("Silakan isi bidang yang diperlukan terlebih dahulu untuk menghasilkan tautan berlangganan");
        return false;
      }

      const url = "surge://install-config?url=";
      window.open(url + this.customSubUrl);
    },
    makeUrl() {
      if (this.form.sourceSubUrl === "" || this.form.clientType === "") {
        this.$message.error("Tautan berlangganan dan klien diperlukan");
        return false;
      }

      let backend =
        this.form.customBackend === ""
          ? defaultBackend
          : this.form.customBackend;

      let sourceSub = this.form.sourceSubUrl;
      sourceSub = sourceSub.replace(/(\n|\r|\n\r)/g, "|");

      this.customSubUrl =
        backend +
        "target=" +
        this.form.clientType +
        "&url=" +
        encodeURIComponent(sourceSub) +
        "&insert=" +
        this.form.insert;

      if (this.advanced === "2") {
        if (this.form.remoteConfig !== "") {
          this.customSubUrl +=
            "&config=" + encodeURIComponent(this.form.remoteConfig);
        }
        if (this.form.excludeRemarks !== "") {
          this.customSubUrl +=
            "&exclude=" + encodeURIComponent(this.form.excludeRemarks);
        }
        if (this.form.includeRemarks !== "") {
          this.customSubUrl +=
            "&include=" + encodeURIComponent(this.form.includeRemarks);
        }
        if (this.form.filename !== "") {
          this.customSubUrl +=
            "&filename=" + encodeURIComponent(this.form.filename);
        }
        if (this.form.appendType) {
          this.customSubUrl +=
            "&append_type=" + this.form.appendType.toString();
        }

        this.customSubUrl +=
          "&emoji=" +
          this.form.emoji.toString() +
          "&list=" +
          this.form.nodeList.toString() +
          "&tfo=" +
          this.form.tfo.toString() +
          "&scv=" +
          this.form.scv.toString() +
          "&fdn=" +
          this.form.fdn.toString() +
          "&sort=" +
          this.form.sort.toString();

        if (this.needUdp) {
          this.customSubUrl += "&udp=" + this.form.udp.toString()
        }

        if (this.form.tpl.surge.doh === true) {
          this.customSubUrl += "&surge.doh=true";
        }

        if (this.form.clientType === "clash") {
          if (this.form.tpl.clash.doh === true) {
            this.customSubUrl += "&clash.doh=true";
          }

          this.customSubUrl += "&new_name=" + this.form.new_name.toString();
        }
      }

      this.$copyText(this.customSubUrl);
      this.$message.success("Langganan khusus disalin ke papan klip");
    },
    makeShortUrl() {
      if (this.customSubUrl === "") {
        this.$message.warning("Silakan buat tautan berlangganan terlebih dahulu, lalu dapatkan tautan pendek yang sesuai");
        return false;
      }

      this.loading = true;

      let data = new FormData();
      data.append("longUrl", btoa(this.customSubUrl));

      this.$axios
        .post(shortUrlBackend, data, {
          header: {
            "Content-Type": "application/form-data; charset=utf-8"
          }
        })
        .then(res => {
          if (res.data.Code === 1 && res.data.ShortUrl !== "") {
            this.curtomShortSubUrl = res.data.ShortUrl;
            this.$copyText(res.data.ShortUrl);
            this.$message.success("Tautan pendek disalin ke papan klip");
          } else {
            this.$message.error("Gagal mendapatkan tautan pendek：" + res.data.Message);
          }
        })
        .catch(() => {
          this.$message.error("Gagal mendapatkan tautan pendek");
        })
        .finally(() => {
          this.loading = false;
        });
    },
    notify() {
      const h = this.$createElement;

      this.$notify({
        title: "Pemberitahuan Privasi",
        type: "warning",
        message: h(
          "i",
          { style: "color: teal" },
          "Berbagai tautan berlangganan (kecuali layanan tautan pendek) menghasilkan implementasi front-end murni tanpa masalah privasi. Layanan konversi back-end disediakan secara default, dan mereka yang peduli dengan privasi harus membangun layanan back-end mereka sendiri."
         )
      });
    },
    confirmUploadConfig() {
      if (this.uploadConfig === "") {
        this.$message.warning("Konfigurasi jarak jauh tidak boleh kosong");
        return false;
      }

      this.loading = true;

      let data = new FormData();
      data.append("password", this.uploadPassword);
      data.append("config", this.uploadConfig);

      this.$axios
        .post(configUploadBackend, data, {
          header: {
            "Content-Type": "application/form-data; charset=utf-8"
          }
        })
        .then(res => {
          if (res.data.code === 0 && res.data.data.url !== "") {
            this.$message.success(
              "Unggahan konfigurasi jarak jauh berhasil, tautan konfigurasi telah disalin ke clipboard, dan masa berlaku adalah tiga bulan."
            );

            // 自动填充至『表单-远程配置』
            this.form.remoteConfig = res.data.data.url;
            this.$copyText(this.form.remoteConfig);

            this.dialogUploadConfigVisible = false;
          } else {
            this.$message.error("Unggahan konfigurasi jarak jauh gagal: " + res.data.msg);
          }
        })
        .catch(() => {
          this.$message.error("Unggahan konfigurasi jarak jauh gagal");
        })
        .finally(() => {
          this.loading = false;
        });
    },
    backendSearch(queryString, cb) {
      let backends = this.options.backendOptions;

      let results = queryString
        ? backends.filter(this.createFilter(queryString))
        : backends;

      // 调用 callback 返回建议列表的数据
      cb(results);
    },
    createFilter(queryString) {
      return candidate => {
        return (
          candidate.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0
        );
      };
    },
    getBackendVersion() {
      this.$axios
        .get(
          defaultBackend.substring(0, defaultBackend.length - 5) + "/version"
        )
        .then(res => {
          this.backendVersion = res.data.replace(/backend\n$/gm, "");
          this.backendVersion = this.backendVersion.replace("subconverter", "");
        });
    },
    saveSubUrl() {
      if (this.form.sourceSubUrl !== '') {
        this.setLocalStorageItem('sourceSubUrl', this.form.sourceSubUrl)
      }
    },
    getLocalStorageItem(itemKey) {
      const now = +new Date()
      let ls = localStorage.getItem(itemKey)

      let itemValue = ''
      if (ls !== null) {
        let data = JSON.parse(ls)
        if (data.expire > now) {
          itemValue = data.value
        } else {
          localStorage.removeItem(itemKey)
        }
      }

      return itemValue
    },
    setLocalStorageItem(itemKey, itemValue) {
      const ttl = process.env.VUE_APP_CACHE_TTL
      const now = +new Date()

      let data = {
        setTime: now,
        ttl: parseInt(ttl),
        expire: now + ttl * 1000,
        value: itemValue
      }
      localStorage.setItem(itemKey, JSON.stringify(data))
    }
  },
};
</script>
