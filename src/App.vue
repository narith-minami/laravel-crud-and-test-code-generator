<template>
  <v-app id="inspire">
    <v-navigation-drawer fixed v-model="drawer" app></v-navigation-drawer>
    <v-toolbar color="indigo" dark fixed app>
      <!--
        <v-toolbar-side-icon @click.stop="drawer = !drawer"></v-toolbar-side-icon>
      -->
      <v-toolbar-title>Laravel CRUD & TestCase Generator</v-toolbar-title>
    </v-toolbar>

    <v-container grid-list-md>
      <v-layout justify-space-between row fill-height>
        <v-flex xs4>
          <v-content>
            <v-form>
              <v-text-field
                label="Model Name (ex.Post)"
                type="text"
                :success="modelName !== ''"
                v-model="modelName"
              ></v-text-field>
              <v-text-field
                label="Key Column (for update query)"
                type="text"
                :success="updateKeyName !== ''"
                v-model="updateKeyName"
              ></v-text-field>
              <v-text-field
                label="Key Column (for select query)"
                type="text"
                :success="selectKeyName !== ''"
                v-model="selectKeyName"
              ></v-text-field>
              <v-layout
                v-if="0 < inputFields.length"
                v-for="(field, index) in inputFields"
                :key="index"
              >
                <v-text-field
                  :readonly="true"
                  :disabled="true"
                  class="pr-2"
                  :label="'No.' + (index + 1).toString()"
                  type="text"
                  :value="field.name"
                ></v-text-field>
                <v-select
                  :readonly="true"
                  :disabled="true"
                  label="Type"
                  solo
                  :items="items"
                  v-model="field.type"
                ></v-select>
              </v-layout>
              <v-layout>
                <v-text-field
                  class="pr-2"
                  label="Column Name"
                  type="text"
                  v-model="inputName"
                ></v-text-field>
                <v-select
                  label="Type"
                  solo
                  :items="items"
                  v-model="selectType"
                ></v-select>
                <v-btn outlline round class="red" @click="addColumn();"
                  >add</v-btn
                >
              </v-layout>
            </v-form>
          </v-content>
        </v-flex>
        <v-flex xs8>
          <v-content>
            <h2>Result (Source Code)</h2>
            <v-textarea
              box
              auto-grow
              name="input-7-4"
              label="Service"
              :value="inputServiceCode"
            ></v-textarea>
            <p>Note:</p>
            <p>1. You need to add "use \App\{{ modelName }};" on Your Class</p>
            <p>2. You need to add $fillabel to {{ modelName }}.php</p>
            <p> -「{{ fillable }}」</p>
            <v-textarea
              box
              auto-grow
              name="input-7-4"
              label="TestCase"
              :value="inputTestCode"
            ></v-textarea>
            <p>Preparation:</p>
            <p>
              - You can create Test Class by 「php artisan make:test
              {{ modelName }}Test --unit」 command
            </p>
            <p>Note:</p>
            <p>
              1. You need to add "use \App\{{ modelName }};" on Your Test Class
            </p>
            <p>
              2. You need to add "use \App\Services\{Your Service Class} on Your
              Test Class"
            </p>
            <p>
              3. You need to add field "private ${Your Service Class} on Your
              Test Class"
            </p>
          </v-content>
        </v-flex>
      </v-layout>
    </v-container>
    <v-footer color="indigo" app inset>
      <span class="white--text text-xs-center"
        ><a
          target="_blank"
          class="white--text text-xs-center"
          src="https://twitter.com/NARI_Creator"
          >&copy;NARI TECH</a
        ></span
      >
    </v-footer>
  </v-app>
</template>

<script>
export default {
  data: () => ({
    drawer: null,
    inputName: "",
    selectType: "",
    modelName: "",
    items: ["integer", "string", "timestamp"],
    updateKeyName: "",
    selectKeyName: "",
    inputFields: [],
    fillable: "",
    dummy: [
      { name: "title", type: "integer" },
      { name: "body", type: "string" }
    ]
  }),
  methods: {
    addColumn: function() {
      if (this.inputName === "") {
        alert("Please input column name.");
        return;
      }
      if (this.selectType === "") {
        alert("Please select type.");
        return;
      }
      const data = { name: this.inputName, type: this.selectType };
      this.inputFields.push(data);
      this.fillable = "protected $fillable = [";
      for (let i = 0; i < this.inputFields.length - 1; ++i) {
        this.fillable += "'" + this.inputFields[i]['name'] + "', ";
      }
      this.fillable += "'" + this.inputFields[this.inputFields.length - 1]['name'] + "'];";
      this.inputName = "";
      this.selectType = "";
    },
    generateCreateMethod: function() {
      let result = "";
      let args = "";
      const data = this.inputFields;
      for (let i = 0; i < data.length - 1; ++i) {
        args += "$" + data[i]["name"] + ", ";
      }
      args += "$" + data[data.length - 1]["name"];
      result +=
        "public function create" + this.modelName + "(" + args + ")" + "\n";
      result += "{" + "\n";
      result += "\t$model = new " + this.modelName + "();" + "\n";
      for (let i = 0; i < data.length; ++i) {
        result +=
          "\t$model->" +
          data[i]["name"] +
          " = $" +
          data[i]["name"] +
          ";" +
          "\n";
      }
      result += "\t$model->save();" + "\n";
      result += "}" + "\n";
      return result;
    },
    generateUpdateMethod: function() {
      let result = "";
      let args = "$" + this.updateKeyName + ", ";
      const data = this.inputFields;
      for (let i = 0; i < data.length - 1; ++i) {
        args += "$" + data[i]["name"] + ", ";
      }
      args += "$" + data[data.length - 1]["name"];
      result +=
        "public function update" + this.modelName + "(" + args + ")" + "\n";
      result += "{" + "\n";
      result +=
        "\t" + "$model = " + this.modelName + "::updateOrCreate(" + "\n";
      result +=
        "\t" +
        "\t" +
        "['" +
        this.updateKeyName +
        "' => $" +
        this.updateKeyName +
        "]," +
        "\n";
      result += "\t" + "\t" + "[";
      for (let i = 0; i < data.length - 1; ++i) {
        result += "'" + data[i]["name"] + "' => $" + data[i]["name"] + ", ";
      }
      result +=
        "'" +
        data[data.length - 1]["name"] +
        "' => $" +
        data[data.length - 1]["name"] +
        "]);" +
        "\n";
      result += "\t" + "return $model;" + "\n";
      result += "}" + "\n";
      return result;
    },
    generateGetMethod: function() {
      let result = "";
      result +=
        "public function get" +
        this.modelName +
        "($" +
        this.selectKeyName +
        ")" +
        "\n";
      result += "{" + "\n";
      result +=
        "\t" +
        "$model = " +
        this.modelName +
        "::where('" +
        this.selectKeyName +
        "', $" +
        this.selectKeyName +
        ")->first();" +
        "\n";
      result += "\t" + "return $model;" + "\n";
      result += "}" + "\n";
      return result;
    },
    getTestParameterCode: function(addText) {
      let inputParamCode =
        "\t$" +
        (addText ? this.updateKeyName : this.selectKeyName) +
        " = 9999;\n";
      const data = this.inputFields;
      for (let i = 0; i < data.length; ++i) {
        const columnName = data[i]["name"];
        const type = data[i]["type"];
        if (type === "integer") {
          const value = addText ? 100 + i : 10 + i;
          inputParamCode += "\t$" + columnName + " = " + value + ";\n";
        } else if (type === "string") {
          const value = addText ? columnName + addText : columnName;
          inputParamCode +=
            "\t$" + columnName + " = 'This is " + value + "';\n";
        } else if (type === "timestamp") {
          inputParamCode += "\t$" + columnName + " = Carbon::now();\n";
        }
      }
      return inputParamCode;
    },
    getTestAssertCode: function() {
      let result = "";
      const data = this.inputFields;
      for (let i = 0; i < data.length; ++i) {
        const columnName = data[i]["name"];
        result +=
          "\t$this->assertSame($model->" +
          columnName +
          ", $" +
          columnName +
          ");\n";
      }
      return result;
    },
    generateTestSetupCode: function() {
      let result = "public function setUp()\n";
      result += "{\n";
      result += "\t"+"$this->DummyService = new \App\Services\DummyService();"+"\n".replace('Dummy', this.modelName)
      result += "\tparent::setUp();";
      result += "}\n";
      return result
    },
    generateUpdateTest: function() {
      let result = "";
      let args = "$" + this.updateKeyName + ", ";
      const data = this.inputFields;
      for (let i = 0; i < data.length - 1; ++i) {
        const columnName = data[i]["name"];
        args += "$" + columnName + ", ";
      }
      args += "$" + data[data.length - 1]["name"];
      result += "public function testUpdate" + this.modelName + "()" + "\n";
      result += "{" + "\n";
      result += this.getTestParameterCode("[update]") + "\n";
      const sName =
        this.modelName.charAt(0).toLowerCase() + this.modelName.slice(1);
      result +=
        "\t$result = $this->" +
        sName +
        "Service->update" +
        this.modelName +
        "(" +
        args +
        ");" +
        "\n";
      result +=
        "\t" +
        "$model = " +
        this.modelName +
        "::where('" +
        this.selectKeyName +
        "', $" +
        this.selectKeyName +
        ")->first();" +
        "\n";
      result += "\n";
      result += this.getTestAssertCode();
      result += "}" + "\n";
      return result;
    },
    generateCreateTest: function() {
      let result = "";
      let args = "";
      const data = this.inputFields;
      for (let i = 0; i < data.length - 1; ++i) {
        const columnName = data[i]["name"];
        args += "$" + columnName + ", ";
      }
      args += "$" + data[data.length - 1]["name"];
      result += "public function testCreate" + this.modelName + "()" + "\n";
      result += "{" + "\n";
      result += this.getTestParameterCode() + "\n";
      const sName =
        this.modelName.charAt(0).toLowerCase() + this.modelName.slice(1);
      result +=
        "\t$result = $this->" +
        sName +
        "Service->create" +
        this.modelName +
        "(" +
        args +
        ");" +
        "\n";
      result +=
        "\t" +
        "$model = " +
        this.modelName +
        "::where('" +
        this.selectKeyName +
        "', $" +
        this.selectKeyName +
        ")->first();" +
        "\n";
      result += "\n";
      result += this.getTestAssertCode();
      result += "}" + "\n";
      return result;
    },
    generateGetTest: function() {
      let result = "";
      let args = "$" + this.selectKeyName;
      const data = this.inputFields;
      result += "public function testGet" + this.modelName + "()" + "\n";
      result += "{" + "\n";
      result += this.getTestParameterCode() + "\n";
      const sName =
        this.modelName.charAt(0).toLowerCase() + this.modelName.slice(1);
      result +=
        "\t$model = $this->" +
        sName +
        "Service->get" +
        this.modelName +
        "(" +
        args +
        ");" +
        "\n";
      result += "\n";
      result += this.getTestAssertCode();
      result += "}" + "\n";
      return result;
    },
    validateInputs: function() {
      if (this.modelName === "") {
        return "Please input Model.";
      }
      if (this.selectKeyName === "" || this.updateKeyName === "") {
        return "Please input select / update key name";
      }
      if (this.inputFields.length === 0) {
        return "Please input Columns fields.";
      }
      return "";
    }
  },
  computed: {
    inputServiceCode: function() {
      const error = this.validateInputs();
      if (error !== "") {
        return error;
      }
      let result = "";
      result += this.generateCreateMethod();
      result += "\n";
      result += this.generateUpdateMethod();
      result += "\n";
      result += this.generateGetMethod();
      return result;
    },
    inputTestCode: function() {
      const error = this.validateInputs();
      if (error !== "") {
        return error;
      }
      let result = "";
      result += this.generateTestSetupCode();
      result += "\n";
      result += this.generateCreateTest();
      result += "\n";
      result += this.generateGetTest();
      result += "\n";
      result += this.generateUpdateTest();
      return result;
    }
  },
  props: {
    source: String
  }
};
</script>
