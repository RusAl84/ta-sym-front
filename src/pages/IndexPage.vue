<template>
  <q-page>
    <q-card class="chatmessages">
      <div class="flexrow">
        <div>
          <q-card-section>
            <div class="text-h6">оценки схожести текстов с целью установления авторства:</div>
          </q-card-section>
          <q-input
            class="inputtext"
            v-model="proc_text"
            type="textarea"
            autogrow
            float-label="Введите текст для анализа"
          />
          <q-card-section>
            <q-btn color="primary" label="Поиск" @click="onProc" />
          </q-card-section>
        </div></div
    ></q-card>
    <q-card class="chatmessages">
      <div class="flexrow">
        <div>
          <q-card-section>
            <div class="text-h6">
              Найденные целевые сообщения:
            </div>
          </q-card-section>
          <q-card-section class="q-gutter-lg">
            <q-btn color="primary" label="Поиск" @click="onFindCL" />
          </q-card-section>
        </div>
      </div>
      <q-card-section v-if="ae_data.length > 0">
        <div v-for="(line, i) in ae_data" :key="i">
          <div>
            <b>Найденный фрагмент &nbsp; {{ i + 1 }}</b>
          </div>
          <div><b>Исходный текст сообщения:</b> {{ line.text }}</div>
          <br />
        </div> </q-card-section
    ></q-card>
    <q-card class="chatmessages">
      <div class="flexrow">
        <div>
          <q-card-section>
            <div class="text-h6">Загрузка данных:</div>
          </q-card-section>
        </div>
        <div>
          <q-card-section>
            <q-uploader
              :url="hostae_uploadae"
              label="Загрузите данные .json/*"
              square
              flat
              bordered
              single
              @uploaded="fileUploaded"
              accept=".json, chatMessages/*"
              style="min-width: 900px; max-width: 900px"
            />
          </q-card-section>
        </div>
      </div>
      <q-card-section>
        <div class="editorclass">
          <q-editor
            v-model="chatmessages"
            max-height="300px"
            :dense="$q.screen.lt.md"
            :toolbar="[
              [
                {
                  label: $q.lang.editor.align,
                  icon: $q.iconSet.editor.align,
                  fixedLabel: true,
                  list: 'only-icons',
                  options: ['left', 'center', 'right', 'justify'],
                },
                {
                  label: $q.lang.editor.align,
                  icon: $q.iconSet.editor.align,
                  fixedLabel: true,
                  options: ['left', 'center', 'right', 'justify'],
                },
              ],
              [
                'bold',
                'italic',
                'strike',
                'underline',
                'subscript',
                'superscript',
              ],
              ['token', 'hr', 'link', 'custom_btn'],
              ['print', 'fullscreen'],
              [
                {
                  label: $q.lang.editor.formatting,
                  icon: $q.iconSet.editor.formatting,
                  list: 'no-icons',
                  options: ['p', 'h1', 'h2', 'h3', 'h4', 'h5', 'h6', 'code'],
                },
                {
                  label: $q.lang.editor.fontSize,
                  icon: $q.iconSet.editor.fontSize,
                  fixedLabel: true,
                  fixedIcon: true,
                  list: 'no-icons',
                  options: [
                    'size-1',
                    'size-2',
                    'size-3',
                    'size-4',
                    'size-5',
                    'size-6',
                    'size-7',
                  ],
                },
                {
                  label: $q.lang.editor.defaultFont,
                  icon: $q.iconSet.editor.font,
                  fixedIcon: true,
                  list: 'no-icons',
                  options: [
                    'default_font',
                    'arial',
                    'arial_black',
                    'comic_sans',
                    'courier_new',
                    'impact',
                    'lucida_grande',
                    'times_new_roman',
                    'verdana',
                  ],
                },
                'removeFormat',
              ],
              ['quote', 'unordered', 'ordered', 'outdent', 'indent'],

              ['undo', 'redo'],
              ['viewsource'],
            ]"
            :fonts="{
              arial: 'Arial',
              arial_black: 'Arial Black',
              comic_sans: 'Comic Sans MS',
              courier_new: 'Courier New',
              impact: 'Impact',
              lucida_grande: 'Lucida Grande',
              times_new_roman: 'Times New Roman',
              verdana: 'Verdana',
            }"
          />
        </div>
      </q-card-section>
    </q-card>
    <q-card class="chatmessages">
      <div class="flexrow">
        <div>
          <q-card-section>
            <div class="text-h6">Загрузка модели:</div>
          </q-card-section>
        </div>
        <div>
          <q-card-section>
            <q-uploader
              :url="hostae_uploadaem"
              label="Загрузите модель векторной семантики .model/*"
              square
              flat
              bordered
              single
              @uploaded="fileUploadedm"
              accept="* /.model"
              style="min-width: 900px; max-width: 900px"
            />
          </q-card-section>
        </div>
      </div>
    </q-card>
  </q-page>
</template>

<script>
import { defineComponent, ref } from "vue";
import axios from "axios";
import * as cfg from "../config.js";

export default {
  data() {
    return {
      file: "",
      hostae: cfg.hostae,
      hostae_uploadae: cfg.hostae_uploadae,
      hostae_uploadaem: ref("http://192.168.3.70:5000/uploadaem"),
      chatmessages: "",
      proc_text: ref(""),
      ttype: ref(""),
      resp_text: ref(""),
      resp_data: ref(""),
      all_data: ref([{}]),
      ae_data: ref([{}]),
      filename: ref(""),
      options: ["RAKE", "YAKE", "BERT"],
    };
  },
  methods: {
    handleFileUpload() {
      this.file = this.$refs.file.files[0];
    },
    fileUploaded({ files, xhr }) {
      let data = JSON.parse(xhr.response);
      this.chatmessages = data["text"];
      this.filename = data["filename"];
    },
    fileUploadedM({ files, xhr }) {
      console.log('Model Uploaded');
    },
    async onProc() {
      const response = await axios({
        method: "post",
        url: cfg.hostae + "get_pattern",
        data: {
          text: this.proc_text,
        },
      });
      console.log(response);
      this.resp_data = response.data;
      this.resp_text = response.data.print_text;
    },
    async onProcAdd() {
      const response = await axios({
        method: "post",
        url: cfg.hostae + "get_pattern_add",
        data: this.resp_data,
      });
      this.all_data = response.data;
      console.log("this.resp_data");
      console.log(this.all_data);
    },
    async onLoadDB() {
      const response = await axios({
        method: "get",
        url: cfg.hostae + "load_db",
        data: this.resp_data,
      });
      this.all_data = response.data;

      console.log(this.all_data);
    },
    async onClearDB() {
      const response = await axios({
        method: "get",
        url: cfg.hostae + "clear_db",
      });
      console.log(response);
    },
    async onFindCL() {
      const response = await axios({
        method: "post",
        url: cfg.hostae + "findae",
        data: {
          filename: this.filename,
          type: this.ttype,
        },
        headers: {
          "Content-Type": "application/json",
        },
      });
      // console.log(response);
      this.ae_data = response.data;
      console.log("ae_data");
      console.log(this.ae_data);
    },
  },
};
</script>

<style lang="sass">
.page
  padding-bottom: 10px
  padding-top: 10px
  padding-left: 10px
  padding-r: 10px

.chatmessages
  padding: 10px
  margin: 10px
.flexrowЁ
  display: flex
  flex-flow: row wrap
  justify-content: flex-start
.editorclass
  max-height: 500px
.inputtext
  width: 800px
</style>
