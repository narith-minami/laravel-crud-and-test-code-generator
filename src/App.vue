<template>
  <v-app id="inspire">
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
              <v-container fluid>
                <p>{{ radios || 'null' }}</p>
                <v-radio-group v-model="radios" :mandatory="false">
                  <v-radio label="Single Record (1:1)" value="single"></v-radio>
                  <v-radio label="Multi Records (1:n)" value="multi"></v-radio>
                </v-radio-group>
              </v-container>
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
                  ref="input_column_name"
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

              <v-textarea
                box
                auto-grow
                name="input-7-4"
                label="Migration Fileds"
                v-model="textInput"
              ></v-textarea>
              <v-btn outlline round class="blue" @click="addColumnsFromText();"
                >Check</v-btn
              >
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
              label="Controller"
              :value="inputControllerCode"
            ></v-textarea>
            <p>Note:</p>
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
            <p>-「{{ fillable }}」</p>
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

            <h3>For Vue.js Code (Sample)</h3>
            <p>This is sample front side code for confirm your API.</p>
            <v-textarea
              box
              auto-grow
              name="input-7-4"
              label="SampleVue.js"
              :value="inputVueCode"
            ></v-textarea>
            <p>Note:</p>
            <p>* api : window.api = require('axios');</p>

            <h3>Routing (API)</h3>
            <p>You need to add domein to "routes/api.php"</p>
            <v-textarea
              box
              auto-grow
              name="input-7-4"
              label="api.php"
              :value="inputRouteCode"
            ></v-textarea>

            <h3>View file</h3>
            <p>You need to add layout file to "resources/view" directory.</p>
            <v-textarea
              box
              auto-grow
              name="input-7-4"
              label="sample.blade.php"
              :value="inputBladeCode"
            ></v-textarea>

            <h3>Remain Work</h3>
            <ul>
              <li>Create or Edit application js</li>
              <li>Add components to webpack.mix.js</li>
              <li>Command: npm run dev</li>
            </ul>
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
          > &copy;NARI TECH</a
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
    items: [
      "integer",
      "double",
      "boolean",
      "string",
      "text",
      "date",
      "dateTime",
      "timestamp"
    ],
    updateKeyName: "",
    selectKeyName: "",
    radios: "single",
    inputFields: [],
    textInput: "",
    fillable: "",
    serviceName: 2 < this.modelName.length ? this.modelName.charAt(0).toLowerCase() + this.modelName.slice(1) : '';
    dummy: [
      { name: "title", type: "integer" },
      { name: "body", type: "string" }
    ]
  }),
  methods: {
    addColumnsFromText: function() {
      if (this.textInput === "") {
        return;
      }
      let migrationFields = this.textInput.replace(/\r\n|\r/g, "\n");
      var lines = migrationFields.split("\n");
      for (var i = 0; i < lines.length; i++) {
        const str = lines[i];
        if (str == "") {
          continue;
        }
        let type = str.split(">")[1].split("(")[0];
        let column = str.split("'")[1];
        if (!type || !column) {
          continue;
        }
        if (!(column === "id" && type === "increments")) {
          const data = { name: column, type: type };
          this.inputFields.push(data);
        }
      }
      this.fillable = "protected $fillable = [";
      for (let i = 0; i < this.inputFields.length - 1; ++i) {
        this.fillable += "'" + this.inputFields[i]["name"] + "', ";
      }
      this.fillable +=
        "'" + this.inputFields[this.inputFields.length - 1]["name"] + "'];";
    },
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
        this.fillable += "'" + this.inputFields[i]["name"] + "', ";
      }
      this.fillable +=
        "'" + this.inputFields[this.inputFields.length - 1]["name"] + "'];";
      this.inputName = "";
      this.selectType = "";
      this.$nextTick(() => {
        this.$refs["input_column_name"].focus();
      });
    },
    generateControllerConstructor: function() {
      const sName =
        this.modelName.charAt(0).toLowerCase() + this.modelName.slice(1);
      let result = "public function __construct(" + this.modelName + "Service $" + sName + "Service)\n";
      result += "{\n";
      result += "\t" + "$this->" + sName + "Service = $" + sName + "Service;" + "\n";
      result += "\t$this->middleware('auth');\n";
      result += "}\n";
      return result;
    },
    generateControllerCreateMethod: function() {
      let result = "";
      let args = "";
      const data = this.inputFields;
      if (this.radios === 'multi') {
        result += "public function create" + this.modelName + "s()" + "\n";
        result += "{" + "\n";
        result += "\t$this->middleware('auth');" + "\n";
        result += "\t// $user_id = \Auth::user()->id; // if you need login userId" + "\n";
        result += "\t$params = request('list_param'); // TODO" + "\n";
        result += "\t$this->" + this.serviceName + "Service->create" + this.modelName + "s($params)" + "\n";
        result += "\treturn ['code' => '200']; // TODO" + "\n";
        result += "}" + "\n";
        return result;
      }
      for (let i = 0; i < data.length - 1; ++i) {
        args += "$" + data[i]["name"] + ", ";
      }
      args += "$" + data[data.length - 1]["name"];
      result += "public function create" + this.modelName + "()" + "\n";
      result += "{" + "\n";
      result += "\t$this->middleware('auth');" + "\n";
      result += "\t// $user_id = \Auth::user()->id; // if you need login userId" + "\n";
      for (let i = 0; i < data.length; ++i) {
        const column = data[i]["name"]
        result += "\t$" + column + " = request('" + this.fSnakeToCamel(column) + "');" + "\n";
      }
      result += "\t$this->" + this.serviceName + "Service->create" + this.modelName + "(" + args + ")" + "\n";
      result += "\treturn ['code' => '200']; // TODO" + "\n";
      result += "}" + "\n";
      return result;
    },
    generateControllerUpdateMethod: function() {
      let result = "";
      let args = "$" + this.updateKeyName + ", ";
      const data = this.inputFields;
      if (this.radios === 'multi') {
        result += "public function update" + this.modelName + "s()" + "\n";
        result += "{" + "\n";
        result += "\t$this->middleware('auth');" + "\n";
        result += "\t// $user_id = \Auth::user()->id; // if you need login userId" + "\n";
        result += "\t$" + column + " = request('" + this.fSnakeToCamel(this.selectKeyName) + "');" + "\n";
        result += "\t$params = request('list_param'); // TODO" + "\n";
        result += "\t$this->" + this.serviceName + "Service->update" + this.modelName + "s($" + this.updateKeyName + ", $params)" + "\n";
        result += "\treturn ['code' => '200']; // TODO" + "\n";
        result += "}" + "\n";
        return result;
      }
      for (let i = 0; i < data.length - 1; ++i) {
        args += "$" + data[i]["name"] + ", ";
      }
      args += "$" + data[data.length - 1]["name"];
      result += "public function update" + this.modelName + "()" + "\n";
      result += "{" + "\n";
      result += "\t$this->middleware('auth');" + "\n";
      result += "\t// $user_id = \Auth::user()->id; // if you need login userId" + "\n";
      result += "\t$" + this.updateKeyName + " = request('" + this.fSnakeToCamel(this.updateKeyName) + "');" + "\n";
      for (let i = 0; i < data.length; ++i) {
        const column = data[i]["name"]
        result += "\t$" + column + " = request('" + this.fSnakeToCamel(column) + "');" + "\n";
      }
      result += "\t$this->" + this.serviceName + "Service->update" + this.modelName + "(" + args + ")" + "\n";
      result += "\treturn ['code' => '200']; // TODO" + "\n";
      result += "}" + "\n";
      return result;
    },
    generateControllerGetMethod: function() {
      let result = "";
      if (this.radios === 'multi') {
        result += "public function get" + this.modelName + "s($" + this.fSnakeToCamel(this.selectKeyName) + ")\n";
        result += "{" + "\n";
        result += "\t$data = $this->" + this.serviceName + "Service->get" + this.modelName + "s(" + this.fSnakeToCamel(this.selectKeyName) + ")" + "\n";
        result += "\treturn ['code' => '200', 'data' => $data]; // TODO" + "\n";
        result += "}" + "\n";
      }
      result += "public function get" + this.modelName + "($" + this.fSnakeToCamel(this.selectKeyName) + ")\n";
      result += "{" + "\n";
      result += "\t$data = $this->" + this.serviceName + "Service->get" + this.modelName + "(" + this.fSnakeToCamel(this.selectKeyName) + ")" + "\n";
      result += "\treturn ['code' => '200', 'data' => $data]; // TODO" + "\n";
      result += "}" + "\n";
      return result;
    },
    generateCreateMethod: function() {
      let result = "";
      let args = "";
      const data = this.inputFields;
      if (this.radios === 'multi') {
        result += "public function create" + this.modelName + "s($params)" + "\n";
        result += "{" + "\n";
        result += "\tforeach ($params as $key => $value) {\n";
        result += "\t\t$model = new " + this.modelName + "();" + "\n";
        for (let i = 0; i < data.length; ++i) {
          const columnName = data[i]["name"];
          result += "\t\t$model->" + columnName + " = value['" + columnName + "'];" + "\n";
        }
        result += "\t\t$model->save();" + "\n";
        result += "\t}" + "\n";
        result += "\treturn true;" + "\n";
        result += "}" + "\n";
        return result;
      }
      for (let i = 0; i < data.length - 1; ++i) {
        args += "$" + data[i]["name"] + ", ";
      }
      args += "$" + data[data.length - 1]["name"];
      result += "public function create" + this.modelName + "(" + args + ")" + "\n";
      result += "{" + "\n";
      result += "\t$model = new " + this.modelName + "();" + "\n";
      for (let i = 0; i < data.length; ++i) {
        result += "\t$model->" + data[i]["name"] + " = $" + data[i]["name"] + ";" + "\n";
      }
      result += "\t$model->save();" + "\n";
      result += "}" + "\n";
      return result;
    },
    generateUpdateMethod: function() {
      let result = "";
      let args = "$" + this.updateKeyName + ", ";
      const data = this.inputFields;
      if (this.radios === 'multi') {
        result += "public function update" + this.modelName + "s($" + this.updateKeyName + ", $params)" + "\n";
        result += "{" + "\n";
        result += "\tforeach ($params as $key => $value) {\n";
        result += "\t\t" + this.modelName + "::updateOrCreate(" + "\n";
        result += "\t\t\t" + "['" + this.updateKeyName + "' => $" + this.updateKeyName + "]," + "\n";
        result += "\t\t\t" + "[";
        for (let i = 0; i < data.length - 1; ++i) {
          const column = data[i]["name"];
          result += "'" + column + "' => $" + column + ", ";
        }
        const lastColumn = data[data.length - 1]["name"];
        result += "'" + lastColumn + "' => $" + lastColumn + "]);" + "\n";
        result += "\t}" + "\n";
        result += "\treturn true;" + "\n";
        result += "}" + "\n";
        return result;
      }

      for (let i = 0; i < data.length - 1; ++i) {
        args += "$" + data[i]["name"] + ", ";
      }
      args += "$" + data[data.length - 1]["name"];
      result +=
        "public function update" + this.modelName + "(" + args + ")" + "\n";
      result += "{" + "\n";
      result += "\t" + "$model = " + this.modelName + "::updateOrCreate(" + "\n";
      result += "\t" + "\t" + "['" + this.updateKeyName + "' => $" + this.updateKeyName + "]," + "\n";
      result += "\t" + "\t" + "[";
      for (let i = 0; i < data.length - 1; ++i) {
        const column = data[i]["name"];
        result += "'" + column + "' => $" + column + ", ";
      }
      const lastColumn = data[data.length - 1]["name"];
      result += "'" + lastColumn + "' => $" + lastColumn + "]);" + "\n";
      result += "\t" + "return $model;" + "\n";
      result += "}" + "\n";
      return result;
    },
    generateGetMethod: function() {
      let result = "";
      if (this.radios === 'multi') {
        result += "public function get" + this.modelName + "s($" + this.selectKeyName + ")\n";
        result += "{" + "\n";
        result += "\t$models = " + this.modelName + "::where('" + this.selectKeyName + "', $" + this.selectKeyName + ")->get();\n";
        result += "\t" + "return $models;" + "\n";
        result += "}" + "\n";
        return result;
      }
      result += "public function get" + this.modelName + "($" + this.selectKeyName + ")\n";
      result += "{" + "\n";
      result += "\t$model = " + this.modelName + "::where('" + this.selectKeyName + "', $" + this.selectKeyName + ")->first();\n";
      result += "\t" + "return $model;" + "\n";
      result += "}" + "\n";
      return result;
    },
    getTestMultiParameterCode: function(addText) {
      let inputParamCode = "";
      const keyColumn = (addText ? this.updateKeyName : this.selectKeyName);
      if (addText) {
        inputParamCode += "\t$results = " + this.modelName + "::where('" + keyColumn + "', 99999)->get();\n";
        inputParamCode += "\t$resultRecord_1 = $results[0];\n";
        inputParamCode += "\t$resultRecord_2 = $results[1];\n\n";
      }
      inputParamCode = "\t$" + keyColumn + " = 9999;\n";

      for (let i = 1; i < 3; i++) {
        inputParamCode += this.getTestArrayParameterCode(i, addText);
      }
      return inputParamCode;
    },
    getTestArrayParameterCode: function(index, addText) {
      let inputParamCode = "";

      if (addText) {
        inputParamCode += "\t$data"+index+" = ['id'=>$resultRecord_"+index+"->id, ";
      } else {
        inputParamCode += "\t$data"+index+" = [";
      }
      const data = this.inputFields;
      for (let i = 0; i < data.length-1; ++i) {
        const columnName = data[i]["name"];
        const type = data[i]["type"];
        inputParamCode += "'"+columnName+"'=>";

        if (type === "integer") {
          const value = addText ? 100 + i : 10 + i;
          inputParamCode += value + index;
        } else if (type === "double") {
          const value = addText ? 100.123 + i : 10.123 + i;
          inputParamCode += value;
        } else if (type === "string" || type === "text") {
          const value = addText ? columnName + addText : columnName;
          inputParamCode += "'This is " + value + index + "'";
        } else if ( type === "date" || type === "dateTime" || type === "timestamp") {
          inputParamCode += "Carbon::now();\n";
        } else if (type === "boolean") {
          const value = addText ? false : true;
          inputParamCode += value;
        }
        inputParamCode += ", ";
      }
      const column = data[data.length-1]["name"];
      inputParamCode += "'"+column+"'=>"+column+"];\n";
      return inputParamCode;
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
        } else if (type === "double") {
          const value = addText ? 100.123 + i : 10.123 + i;
          inputParamCode += "\t$" + columnName + " = " + value + ";\n";
        } else if (type === "string" || type === "text") {
          const value = addText ? columnName + addText : columnName;
          inputParamCode +=
            "\t$" + columnName + " = 'This is " + value + "';\n";
        } else if (
          type === "date" ||
          type === "dateTime" ||
          type === "timestamp"
        ) {
          inputParamCode += "\t$" + columnName + " = Carbon::now();\n";
        } else if (type === "boolean") {
          const value = addText ? false : true;
          inputParamCode += "\t$" + columnName + " = " + value + ";\n";
        }
      }
      return inputParamCode;
    },
    getTestMultiAssertCode: function() {
      let result = "\t$this->assertTrue(count($results) === 2);\n";
      result += "\t$resultRecord_1 = $results[0];\n"
      result += "\t$resultRecord_2 = $results[1];\n"
      const data = this.inputFields;
      for (let i = 0; i < data.length; ++i) {
        const columnName = data[i]["name"];
        result += "\t$this->assertSame($resultRecord_1->" + columnName + ", $data1['" + columnName + "']);\n";
        result += "\t$this->assertSame($resultRecord_2->" + columnName + ", $data2['" + columnName + "']);\n";
      }
      return result;
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
      const sName =
        this.modelName.charAt(0).toLowerCase() + this.modelName.slice(1);
      let result = "public function setUp()\n";
      result += "{\n";
      result +=
        "\t" +
        "$this->" +
        sName +
        "Service = new \\App\\Services\\" +
        this.modelName +
        "Service();" +
        "\n";
      result += "\tparent::setUp();\n";
      result += "}\n";
      return result;
    },
    generateTestTearDownAfterClassCode: function() {
      const sName =
        this.modelName.charAt(0).toLowerCase() + this.modelName.slice(1);
      let result = "public function tearDownAfterClass()\n";
      result += "{\n";
      result += "\t" + this.modelName + "::where('" + this.selectKeyName + "', 99999)->delete();\n";
      result += "}\n";
      return result;
    },
    generateUpdateTest: function() {
      let result = "";

      const data = this.inputFields;
      if (this.radios === 'multi') {
        const args = "$"+this.updateKeyName+", [$data1,$data2]";
        result += "public function testUpdate" + this.modelName + "s()" + "\n";
        result += "{" + "\n";
        result += this.getTestMultiParameterCode('[change]') + "\n";
        const sName = this.modelName.charAt(0).toLowerCase() + this.modelName.slice(1);
        result += "\t$this->" + sName + "Service->update" + this.modelName + "s(" + args + ");\n";
        result += "\t$results = " + this.modelName + "::where('" + this.selectKeyName + "', $" + this.selectKeyName + ")->get();\n";
        result += "\n";
        result += this.getTestMultiAssertCode();
        result += "}" + "\n";
        return result;
      }
      let args = "$" + this.updateKeyName + ", ";
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
      const data = this.inputFields;
      if (this.radios === 'multi') {
        const args = "$"+this.selectKeyName+", [$data1,$data2]";
        result += "public function testCreate" + this.modelName + "s()" + "\n";
        result += "{" + "\n";
        result += this.getTestMultiParameterCode() + "\n";
        const sName = this.modelName.charAt(0).toLowerCase() + this.modelName.slice(1);
        result += "\t$this->" + sName + "Service->create" + this.modelName + "s(" + args + ");\n";
        result += "\t$results = " + this.modelName + "::where('" + this.selectKeyName + "', $" + this.selectKeyName + ")->get();\n";
        result += "\n";
        result += this.getTestMultiAssertCode();
        result += "}" + "\n";
        return result;
      }
      let args = "";
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
      result += "\t$result = $this->" + sName + "Service->create" + this.modelName + "(" + args + ");\n";
      result += "\t$model = " + this.modelName + "::where('" + this.selectKeyName + "', $" + this.selectKeyName + ")->first();\n";
      result += "\n";
      result += this.getTestAssertCode();
      result += "}" + "\n";
      return result;
    },
    generateGetTest: function() {
      let result = "";
      let args = "$" + this.selectKeyName;
      if (this.radios === 'multi') {
        result += "public function testGet" + this.modelName + "s()" + "\n";
        result += "{" + "\n";
        result += this.getTestMultiParameterCode() + "\n";
        const sName = this.modelName.charAt(0).toLowerCase() + this.modelName.slice(1);
        result += "\t$results = $this->" + sName + "Service->get" + this.modelName + "s(" + args + ");\n";
        result += "\n";
        result += this.getTestMultiAssertCode();
        result += "}" + "\n";
        return result;
      }

      result += "public function testGet" + this.modelName + "()" + "\n";
      result += "{" + "\n";
      result += this.getTestParameterCode() + "\n";
      const sName = this.modelName.charAt(0).toLowerCase() + this.modelName.slice(1);
      result += "\t$model = $this->" + sName + "Service->get" + this.modelName + "(" + args + ");\n";
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
    },
    generateVueTemplate: function() {
      let result = "<template>\n";
      result += "<div>\n";
      if (this.radios === 'multi') {
        result += '\t<div v-for="(data, index) in list" :key="index">\n';
        result += '\t<p>[ Record.{{ index }} ]</p>\n';
        for (var i = 0; i < this.inputFields.length; i++) {
          const column = this.inputFields[i].name;
          const seq = i + 1;
          result += "\t\t<span>" + seq + ". " + column + " : {{ data." + column + " }}</span>\n";
        }
        result += "\t</div>\n";
        result += "</div>\n";
        result += "</template>\n";
        return result;
      }
      for (var i = 0; i < this.inputFields.length; i++) {
        const column = this.inputFields[i].name;
        const seq = i + 1;
        result += "\t<span>" + seq + ". " + column + " : {{ " + this.fSnakeToCamel(column) + " }}</span>\n";
      }
      for (var i = 0; i < this.inputFields.length; i++) {
        const column = this.inputFields[i].name;
        const col = this.fSnakeToCamel(column)
        result += '\t<input type="text" v-model="'+col+'" :value="'+col+'">\n'
      }
      result += "</div>\n";
      result += "</template>\n";
      return result;
    },
    fSnakeToCamel: function(snakeStr) {
      return snakeStr.replace(/_./g,
          function(s) {
              return s.charAt(1).toUpperCase();
          }
      );
    },
    fCamelToSnake: function() {

    },
    generateVueScript: function() {
      const keyName = this.fSnakeToCamel(this.selectKeyName)
      const model = this.modelName.toLowerCase();
      let result = "<script>\n"
      result += "\texport default {\n"
      result +=        "\t\tcreated() {\n"
      result +=          "\t\t\tthis.fetch()\n"
      result +=        "\t\t},\n"
      result +=        "\t\tdata() {\n"
      result +=          "\t\t\treturn {\n"
      result +=            "\t\t\t\tlist: [],\n"
      for (var i = 0; i < this.inputFields.length; i++) {
        const column = this.inputFields[i].name
        result += "\t\t\t\t"+this.fSnakeToCamel(column)+": null,\n"
      }
      result +=          "\t\t\t}\n"
      result +=        "\t\t},\n"
      result +=        "\t\tprops: ['" + keyName + "'],\n"
      result +=        "\t\tmethods: {\n"
      result +=          "\t\t\tfetch: function() {\n"
      result +=          "\t\t\t\tapi.get('/api/"+model+"/' + this." +keyName+ ").then(rs => {\n"
      if (this.radios === 'multi') {
        result += "\t\t\t\t\tthis.list = rs.data.data\n"
      } else {
        for (var i = 0; i < this.inputFields.length; i++) {
          const column = this.inputFields[i].name
          result += "\t\t\t\t\tthis."+this.fSnakeToCamel(column)+" = rs.data.data."+column+"\n"
        }
      }
      result +=          "\t\t\t\t})\n"
      result +=          "\t\t\t},\n"
      result +=          "\t\t\tinsert: function() {\n"
      result += "\t\t\t\tconst data = {\n"
      for (var i = 0; i < this.inputFields.length; i++) {
        const column = this.inputFields[i].name
        const isLast = (i === this.inputFields.length-1)
        result += "\t\t\t\t\t"+column+": this."+this.fSnakeToCamel(column)+ (isLast?"\n":",\n")
      }
      result += "\t\t\t\t}\n"
      result +=          "\t\t\t\tapi.post('/api/"+model+"/create', data).then(rs => {\n"
      result += "\t\t\t\t})\n"
      result +=          "\t\t\t}\n"
      result +=        "\t\t}\n"
      result +=      "\t}\n"
      result += "</" + "script>"
      return result
    },
    generateRouting: function() {
      const keyName = this.fSnakeToCamel(this.selectKeyName)
      let result = "";
      const prefix = "Route::middleware('api')->";
      const lowModelStr = this.modelName.toLowerCase();
      // get
      result += prefix + "get('/" + lowModelStr + "/{"+keyName+"}', '" + this.modelName + "Controller@get"+this.modelName+"');\n"
      // insert
      result += prefix + "post('/" + lowModelStr + "/create', '" + this.modelName + "Controller@create"+this.modelName+"');\n"
      return result
    },
    generateBlade: function() {
      const key = this.fSnakeToCamel(this.selectKeyName)
      let result = "";
      const model = this.modelName.toLowerCase();
      result += "@extends('layout.default')\n"
      result += "\n"
      result += "@section('content')\n"
      result += "<"+model+" :"+key+"=“{{ $"+key+" }}”></post>\n"
      result += "\n"
      result += '<!—- *TODO* You add application js -->\n'
      result += "@endsection"
      return result
    }
  },
  computed: {
    inputControllerCode: function() {
      const error = this.validateInputs();
      if (error !== "") {
        return error;
      }
      let result = "";
      result += this.generateControllerConstructor();
      result += "\n";
      result += this.generateControllerCreateMethod();
      result += "\n";
      result += this.generateControllerUpdateMethod();
      result += "\n";
      result += this.generateControllerGetMethod();
      return result;
    },
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
      const sName =
        this.modelName.charAt(0).toLowerCase() + this.modelName.slice(1);
      result += "private $" + sName + "Service;\n";
      result += "\n";
      result += this.generateTestSetupCode();
      result += "\n";
      result += this.generateTestTearDownAfterClassCode();
      result += "\n";
      result += this.generateCreateTest();
      result += "\n";
      result += this.generateGetTest();
      result += "\n";
      result += this.generateUpdateTest();
      return result;
    },
    inputVueCode: function() {
      const error = this.validateInputs();
      if (error !== "") {
        return error;
      }
      let result = "";
      result += this.generateVueTemplate();
      result += "\n";
      result += this.generateVueScript();
      return result;
    },
    inputRouteCode: function() {
      const error = this.validateInputs();
      if (error !== "") {
        return error;
      }
      let result = "";
      result += this.generateRouting();
      return result;
    },
    inputBladeCode: function() {
      const error = this.validateInputs();
      if (error !== "") {
        return error;
      }
      let result = "";
      result += this.generateBlade();
      return result;
    }
  },
  props: {
    source: String
  }
};
</script>
