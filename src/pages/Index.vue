<template>
  <q-page class="q-ma-sm row">
    <div v-if="elSelectedToInsert" class="col-12 q-mb-sm q-gutter-sm">
      <q-btn color="red" icon="delete" @click="deleteComponent"></q-btn>
      <q-btn
        color="red"
        icon="close"
        @click="elSelectedToInsert = null"
      ></q-btn>
    </div>
    <div class="col-md-2">
      <q-list bordered separator>
        <q-item
          v-for="(el, key) in elements"
          :key="key"
          clickable
          v-ripple
          @click="openModal(el)"
        >
          <q-item-section>{{ el.name }}</q-item-section>
        </q-item>
      </q-list>
    </div>
    <div class="col-10 q-pa-sm">
      <iframe
        ref="iframeEhr"
        style="width: 100%; height: calc(100vh - 100px)"
        :srcdoc="htmlText"
        frameborder="0"
      ></iframe>
    </div>
    <q-dialog v-model="modalStateComponent" persistent @before-hide="hideModal">
      <q-card style="width: 700px; max-width: 80vw">
        <q-card-section>
          <div class="text-h6">Configurações de componente</div>
        </q-card-section>

        <q-card-section class="q-pt-none row">
          <div v-if="!elementSelected.noHaveText" class="col-12">
            <q-input v-model="textInput" label="Título"></q-input>
          </div>
          <div
            v-if="
              ['checkbox', 'radio', 'SELECT'].includes(elementSelected.type)
            "
            class="col-12"
          >
            <q-input
              v-for="(oC, key) in optionsIfCheckbox"
              :key="key"
              v-model="oC.value"
              label="Opção"
            >
              <template #after>
                <q-btn
                  icon="add"
                  round
                  color="deep-purple-6"
                  @click="addOption"
                ></q-btn>
              </template>
            </q-input>
          </div>
          <div
            v-if="['H'].includes(elementSelected.type)"
            class="col-12 q-mt-md text-bold"
          >
            <p>Tipos de título:</p>
            <q-btn-toggle
              v-model="typeH"
              push
              glossy
              toggle-color="deep-purple-6"
              :options="[
                { label: 'H1', value: '1' },
                { label: 'H2', value: '2' },
                { label: 'H3', value: '3' },
                { label: 'H4', value: '4' },
                { label: 'H5', value: '5' },
                { label: 'H6', value: '6' },
              ]"
            />
          </div>
          <div v-if="elementSelected.haveSize" class="col-12 q-mt-md text-bold">
            <p>Tamanho de input:</p>
            <q-btn-toggle
              v-model="typeSize"
              push
              glossy
              toggle-color="deep-purple-6"
              :options="[
                { label: 'SM', value: 'sm' },
                { label: 'MD', value: 'md' },
                { label: 'LG', value: 'lg' },
              ]"
            />
          </div>
          <div v-if="elementSelected.isTable" class="row">
            <div class="text-h6">Titulo</div>
            <div class="col-12 q-col-gutter-sm">
              <div
                v-for="(tCC, key) in tableToCreateContent.title"
                :key="key"
                class="col-4"
              >
                <q-input v-model="tCC.value">
                  <template #after>
                    <q-btn
                      icon="add"
                      round
                      color="deep-purple-6"
                      @click="addTitleTable"
                    ></q-btn> </template
                ></q-input>
              </div>
            </div>
          </div>
          <div v-if="elementSelected.isTable" class="row">
            <div class="text-h6">Conteudo</div>
            <div class="col-12 q-col-gutter-sm">
              <div
                v-for="(tCC, key) in tableToCreateContent.content"
                :key="key"
                class="col-4"
              >
                <q-input v-model="tCC.value"> </q-input>
              </div>
            </div>
          </div>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn outline label="cancelar" color="red" v-close-popup />
          <q-btn
            label="INSERIR"
            color="deep-purple-10"
            @click="insertElement()"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script>
import { defineComponent, ref, onMounted } from "vue";
import { uuid } from "vue-uuid";

export default defineComponent({
  name: "PageIndex",
  setup() {
    const iframeEhr = ref(null);

    const showing = ref(false);

    const questionId = ref(0);

    const modalStateComponent = ref(false);

    const elementSelected = ref({});

    const textInput = ref("");

    const typeH = ref("1");

    const typeSize = ref("md");

    const codeLogic = ref([]);

    const elSelectedToInsert = ref(null);

    const tableToCreateContent = ref({
      title: [
        {
          value: "",
        },
      ],
      content: [
        {
          value: "",
        },
      ],
    });

    const optionsIfCheckbox = ref([
      {
        value: "",
      },
    ]);

    const elements = ref({
      input: {
        name: "Campo de texto",
        element: "INPUT",
        type: "text",
        imcrementQuestion: true,
        setAttribute: true,
        haveSize: true,
        generatorElement({ key, title, typeSize }) {
          const xFather = document.createElement("DIV");

          const xPTitle = document.createElement("P");
          xPTitle.textContent = title;

          xFather.appendChild(xPTitle);

          const uuidInput = uuid.v1();

          const x = document.createElement(this.element);

          if (this.setAttribute) {
            x.setAttribute("type", this.type);
          }

          x.setAttribute("class", `form-control form-control-${typeSize}`);

          if (this.imcrementQuestion) {
            key.value++;
            x.setAttribute("id", uuidInput);
            x.setAttribute("name", "${q_" + key.value + "}");
          }

          xFather.appendChild(x);

          return { xFather, uuidInput };
        },
      },
      input2: {
        name: "Múltipla escolha",
        element: "INPUT",
        type: "checkbox",
        imcrementQuestion: true,
        setAttribute: true,
        generatorElement({ key, title, questionsValues }) {
          const xFather = document.createElement("DIV");

          const xPTitle = document.createElement("P");
          xPTitle.textContent = title;

          xFather.appendChild(xPTitle);

          const ids = [];

          for (const check of questionsValues) {
            const xFatherCheck = document.createElement("DIV");
            xFatherCheck.classList.add("form-check");

            key.value++;

            const uuidInput = uuid.v1();

            ids.push(uuidInput);

            let xCheck = document.createElement("INPUT");

            let xLabel = document.createElement("LABEL");
            xLabel.setAttribute("for", uuidInput);
            xLabel.classList.add("form-check-label");
            xLabel.textContent = check.value;

            xCheck.setAttribute("type", this.type);
            xCheck.setAttribute("id", uuidInput);
            xCheck.setAttribute("name", "${q_" + key.value + "}");
            xCheck.classList.add("form-check-input");
            xCheck.setAttribute("value", check.value);

            xFatherCheck.appendChild(xCheck);
            xFatherCheck.appendChild(xLabel);
            xFather.appendChild(xFatherCheck);
          }

          return { xFather, uuidInput: ids };
        },
      },
      input3: {
        name: "Escolha única",
        element: "INPUT",
        type: "radio",
        imcrementQuestion: true,
        setAttribute: true,
        generatorElement({ key, title, questionsValues }) {
          const xFather = document.createElement("DIV");

          const xPTitle = document.createElement("P");
          xPTitle.textContent = title;

          key.value++;

          xFather.appendChild(xPTitle);

          const ids = [];

          for (const check of questionsValues) {
            let uuidInput = uuid.v1();

            ids.push(uuidInput);

            const xFatherRadio = document.createElement("DIV");
            xFatherRadio.classList.add("form-check");

            let xCheck = document.createElement("INPUT");

            let xLabel = document.createElement("LABEL");
            xLabel.classList.add("form-check-label");
            xLabel.setAttribute("for", uuidInput);
            xLabel.textContent = check.value;

            xCheck.setAttribute("type", this.type);
            xCheck.setAttribute("id", uuidInput);
            xCheck.setAttribute("name", "${q_" + key.value + "}");
            xCheck.classList.add("form-check-input");
            xCheck.setAttribute("value", check.value);

            xFatherRadio.appendChild(xCheck);
            xFatherRadio.appendChild(xLabel);
            xFather.appendChild(xFatherRadio);
          }

          return { xFather, uuidInput: ids };
        },
      },
      input4: {
        name: "Titulo",
        element: "H",
        type: "H",
        imcrementQuestion: false,
        setAttribute: false,
        generatorElement({ key, title, typeH }) {
          const xFather = document.createElement("DIV");

          const uuidInput = uuid.v1();

          const xHTitle = document.createElement(`H${typeH}`);
          xHTitle.textContent = title;

          xHTitle.setAttribute("id", uuidInput);

          xFather.appendChild(xHTitle);

          return { xFather, uuidInput };
        },
      },
      input5: {
        name: "Paragrafo",
        element: "P",
        type: "P",
        imcrementQuestion: false,
        setAttribute: false,
        generatorElement({ key, title }) {
          const xFather = document.createElement("DIV");

          const uuidInput = uuid.v1();

          const xHTitle = document.createElement(this.type);
          xHTitle.textContent = title;

          xHTitle.setAttribute("id", uuidInput);

          xFather.appendChild(xHTitle);

          return { xFather, uuidInput };
        },
      },
      input6: {
        name: "Linha",
        element: "HR",
        type: "HR",
        imcrementQuestion: false,
        setAttribute: false,
        noHaveText: true,
        generatorElement({ key, title }) {
          const xFather = document.createElement("DIV");

          const uuidInput = uuid.v1();

          const xHTitle = document.createElement(this.type);
          xHTitle.textContent = title;

          xHTitle.setAttribute("id", uuidInput);

          xFather.appendChild(xHTitle);

          return { xFather, uuidInput };
        },
      },
      input7: {
        name: "Seleção",
        element: "SELECT",
        type: "SELECT",
        imcrementQuestion: true,
        setAttribute: false,
        noHaveText: false,
        haveSize: true,
        generatorElement({ key, title, questionsValues, typeSize }) {
          const xFather = document.createElement("DIV");

          key.value++;

          const uuidInput = uuid.v1();

          const xPTitle = document.createElement("P");
          xPTitle.textContent = title;

          xFather.appendChild(xPTitle);

          const xSELECTTitle = document.createElement(this.type);

          xSELECTTitle.setAttribute("id", uuidInput);

          for (const check of questionsValues) {
            var xOPTION = document.createElement("option");

            xOPTION.setAttribute("value", check.value);

            var t = document.createTextNode(check.value);
            xOPTION.appendChild(t);

            xSELECTTitle.appendChild(xOPTION);
          }

          xSELECTTitle.setAttribute(
            "class",
            `form-control form-control-${typeSize}`
          );

          xSELECTTitle.setAttribute("name", "${q_" + key.value + "}");

          xFather.appendChild(xSELECTTitle);

          return { xFather, uuidInput };
        },
      },
      input8: {
        name: "Tabela",
        element: "TABLE",
        type: "TABLE",
        imcrementQuestion: false,
        setAttribute: false,
        noHaveText: false,
        haveSize: false,
        isTable: true,
        generatorElement({
          key,
          title,
          questionsValues,
          typeSize,
          tableToCreateContent,
        }) {
          const xFather = document.createElement("DIV");

          const ids = [];

          const uuidTable = uuid.v1();
          ids.push(uuidTable);

          var xTABLE = document.createElement("TABLE");
          xTABLE.setAttribute("id", uuidTable);
          xTABLE.classList.add("table");

          const uuidTableTR = uuid.v1();

          ids.push(uuidTableTR);

          var yTR = document.createElement("TR");
          yTR.setAttribute("id", uuidTableTR);

          const uuidTableTRBODY = uuid.v1();

          ids.push(uuidTableTRBODY);

          var xTRBODY = document.createElement("TR");
          xTRBODY.setAttribute("id", uuidTableTRBODY);

          const uuidTableTBODY = uuid.v1();

          ids.push(uuidTableTBODY);

          var xTBODY = document.createElement("TBODY");
          xTBODY.setAttribute("id", uuidTableTBODY);

          const uuidTableTHEAD = uuid.v1();

          ids.push(uuidTableTHEAD);
          var yTHEAD = document.createElement("THEAD");
          yTHEAD.setAttribute("id", uuidTableTHEAD);

          console.log(tableToCreateContent);

          for (const [key, value] of tableToCreateContent.title.entries()) {
            const uuidTableTH = uuid.v1();

            ids.push(uuidTableTH);

            var tTH = document.createElement("TH");
            tTH.setAttribute("id", uuidTableTH);
            tTH.textContent = value.value;

            yTR.appendChild(tTH);

            const uuidTableTRBODY = uuid.v1();

            ids.push(uuidTableTRBODY);

            var tTD = document.createElement("TD");
            tTD.setAttribute("id", uuidTableTRBODY);
            tTD.textContent = tableToCreateContent.content[key].value;
            xTRBODY.appendChild(tTD);
          }

          yTHEAD.appendChild(yTR);

          xTBODY.appendChild(xTRBODY);

          xTABLE.appendChild(yTHEAD);
          xTABLE.appendChild(xTBODY);

          xFather.appendChild(xTABLE);

          console.log(xFather);

          return { xFather, uuidInput: ids };
        },
      },
      input9: {
        name: "Imagem",
        element: "IMG",
        type: "IMG",
        imcrementQuestion: false,
        setAttribute: false,
        noHaveText: false,
        generatorElement({ key, title }) {
          const xFather = document.createElement("DIV");

          const uuidInput = uuid.v1();

          const xHTitle = document.createElement(this.type);
          xHTitle.src = title;
          xHTitle.classList.add("img-fluid");
          xHTitle.setAttribute("id", uuidInput);

          xFather.appendChild(xHTitle);

          return { xFather, uuidInput };
        },
      },
    });

    let htmlText = ref(String.raw`
      <head>
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.3.1/dist/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">

    <title>Hello, world!</title>
  </head>
      <style>
        .colorFocus{
          background-color: LightGray;
        }
      </style>
      <div id="ehr-app"></div>
      `);

    function insertElement() {
      const el = elementSelected.value;

      const { xFather, uuidInput } = el.generatorElement({
        key: questionId,
        title: textInput.value,
        questionsValues: optionsIfCheckbox.value,
        typeH: this.typeH,
        typeSize: this.typeSize,
        tableToCreateContent: tableToCreateContent.value,
      });

      const elGenerated = xFather;

      if (elGenerated) {
        const documentFrame =
          iframeEhr.value.contentDocument || x.contentWindow.document;

        let fatherDiv = null;

        if (elSelectedToInsert.value) {
          fatherDiv = elSelectedToInsert.value;
        } else {
          fatherDiv = documentFrame.getElementById("ehr-app");
        }

        fatherDiv.appendChild(elGenerated);

        const uuidDiv = uuid.v1();

        elGenerated.setAttribute("id", uuidDiv);

        elSelectedToInsert.value = null;

        codeLogic.value.push({
          uuidInput,
          uuidDiv,
        });

        documentFrame.addEventListener("click", (e) => {
          elSelectedToInsert.value = null;

          const elementClicked = documentFrame.getElementById(e.target.id);

          if (elementClicked) {
            elSelectedToInsert.value = elementClicked;
            console.log(elementClicked, elSelectedToInsert.value);
          }
        });
      }

      modalStateComponent.value = false;
    }

    function openModal(el) {
      modalStateComponent.value = true;
      elementSelected.value = el;
    }

    function hideModal() {
      modalStateComponent.value = false;

      elementSelected.value = {};

      textInput.value = "";

      optionsIfCheckbox.value = [
        {
          value: "",
        },
      ];
    }

    function addOption() {
      optionsIfCheckbox.value.push({
        value: "",
      });
    }

    function addTitleTable() {
      tableToCreateContent.value.title.push({ value: "" });
      tableToCreateContent.value.content.push({ value: "" });
    }

    function addContentTable() {
      tableToCreateContent.value.content.push({ value: "" });
    }

    function deleteComponent() {
      elSelectedToInsert.value.parentElement.remove();
      elSelectedToInsert.value = null;
    }

    return {
      modalStateComponent,
      iframeEhr,
      openModal,
      textInput,
      insertElement,
      addOption,
      hideModal,
      showing,
      elements,
      htmlText,
      elementSelected,
      optionsIfCheckbox,
      typeH,
      typeSize,
      codeLogic,
      tableToCreateContent,
      addTitleTable,
      addContentTable,
      elSelectedToInsert,
      deleteComponent,
    };
  },
});
</script>
