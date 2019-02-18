<template>
  <div>
    <h1>Test Job</h1>
    <form class="form" action="#" v-if="form" @submit.prevent="submitForm">
      <ul>
        <li v-for="(error, index) in errors" :key="index">{{error}}</li>
      </ul>
      <table>
        <tr class="row" v-for="(row, index) in layout" :key="index">
        <td class="column" v-for="(fieldName, index) in row" :key="index">
          <div class="field" v-if="fieldName">
            <label>{{form.fields[fieldName].label}}</label>
            <div>
              <component
                :fieldAction="fieldAction"
                v-bind:is="form.fields[fieldName].type === 'checkbox' ? 'input' : form.fields[fieldName].type"
                :type="form.fields[fieldName].type === 'input' ? 'text' : 'checkbox'"
                :value="fields[fieldName]"
                v-on:input="fieldChange(fieldName, form.fields[fieldName].type, $event.target)"
                :checked="form.fields[fieldName].type === 'checkbox' && fields[fieldName]"
                v-on:change="fieldChange(fieldName, form.fields[fieldName].type, $event.target)"
                v-on:click="fieldAction(form.fields[fieldName].behavior)"
                v-show="form.fields[fieldName].visibility === undefined || form.fields[fieldName].visibility"
              >
                <option value="" default></option>
                <option                   
                  v-if="form.fields[fieldName].type === 'select'"
                  v-for="(option, index) in form.fields[fieldName].items"
                  :key="index"
                  :value="option.value"
                >
                {{option.label}}
                </option>
              </component>
            </div>
          </div>
        </td>
      </tr>
      </table>
      <button type="submit">Отправить</button>
    </form>
    <p class="preloader" v-if="!form">Форма загружается</p>
  </div>
</template>

<script>
export default {
  name: "DynamicForm",
  props: {
    api: {
      type: String,
      required: true,      
    },
    id: {
      type: String,
      required: true,      
    }
  },
  data() {
    return {
      form: null,
      layout: null,
      fields: {
        id: null,
        type: null,
        email: null,
        showDescription: false,
        description: null,
      },
      errors: []
    };
  },
  methods: {
    fieldAction(behaviors) {
      if (!Array.isArray(behaviors)) return;
      behaviors.map(behavior => {
        switch(behavior.action) {
          case 'visibility_toggle': {
            this.form.fields[behavior.target].visibility = !this.form.fields[behavior.target].visibility;
            return;
          }
          default: {
            return;
          }
        }
      });
    },
    fieldChange(fieldName, fieldType, target) {
      if (fieldType === 'checkbox') {
        this.fields[fieldName] = target.checked
      } else {
        this.fields[fieldName] = target.value;
      }
    },
    submitForm() {
      this.errors = [];
      Object.keys(this.fields).map(field => {
        const fieldValue = this.fields[field];
        const validators = this.form.fields[field].validators;
        if (validators && Array.isArray(validators)) {
          validators.map(validator => {
            if (validator.required && !fieldValue) {
              this.errors.push(`Поле ${field} обязательно к заполнению`);
            }
            if (validator.type && fieldValue) {
              if (validator.type === 'email' && !/.+@.+\./g.test(fieldValue)) {
                this.errors.push(`Поле ${field} заполнено не верно`);
              }              
            }
          })
        }
      });
      if (!this.errors.length) {
        console.log(JSON.stringify(this.fields));
      }
    }
  },
  mounted() {
    fetch(`${this.api}/${this.id}`)
      .then(response => response.json())
      .then(json => {
        const layoutPairs = Object.entries(json.layout);
        const maxRow = Math.max(...layoutPairs.map(el => el[1].row));
        const maxCol = Math.max(...layoutPairs.map(el => el[1].col));
        this.layout = [];
        for (let i = 0; i < maxRow; i++) {
          this.layout[i] = [];
        }
        for (const [name, coordinates] of layoutPairs) {
          this.layout[coordinates.row - 1][coordinates.col - 1] = name;
        }
        this.form = json;
      });
  }
}
</script>

<style scoped>
table {
  width: 100%;
  border: 1px solid #ccc;
  margin-bottom: 1px;
}
ul {
  color: red;
  text-align: left;
}
td {
  width: 50%;
  padding: 10px;
  border: 1px solid #ccc;
}
h1 {
  margin: 40px 0 20px;
}
.form {
  width: 500px;
  margin: auto;
}
.column {
  flex-grow: 1;
}
</style>
