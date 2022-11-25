<template>
  <div>
    <h1>Hello component</h1>
    <p>{{this.model}}</p>
    <p>{{this.id}}</p>

    <div class="alert alert-danger" role="alert" v-for="(error, index) in errors" :key="index">
      {{ error }}
    </div>

    <div class="panel panel-warning">
      <div class="panel-heading">
        <b class="panel-title arrow-left" style="color: #ac9937">Заявления
          <i v-on:click="isHidden = !isHidden">+</i>
        </b>
      </div>

      <div class="form-group row">
        <div class="col-sm-2">
          <h5>Дата</h5>
          <input type="date" class="form-control" v-on:change="filterDate($event.target.value)">
        </div>

        <div class="col-sm-3">
          <h5>Событие</h5>
          <select class="form-control" v-on:change="filterType($event.target.value)">
            <option value="0" selected>Все</option>
            <option v-for="(type, index) in types" v-bind:key="index" :value="index">{{ type }}</option>
          </select>
        </div>

        <div class="col-sm-2">
          <h5>Статус</h5>
          <select class="form-control" v-on:change="filterStatus($event.target.value)">
            <option value="0" selected>Все</option>
            <option value="NONE">Созданно</option>
            <option value="ACTIVE">Активно</option>
            <option value="DONE">Завершенно</option>
          </select>
        </div>
      </div>

      <table class="table">
        <thead>
        <tr>
          <th scope="col">#</th>
          <th scope="col">Дата</th>
          <th scope="col">Событие</th>
          <th scope="col">Статус</th>
          <th scope="col">Коментарии</th>
          <th scope="col">Фаил</th>
          <th scope="col">Информация</th>
        </tr>
        </thead>
        <tbody>
        <tr style="text-align: left;" v-for="(item, index) in filteredStatements" v-bind:key="index">
          <th scope="row">{{++index}}</th>
          <th>{{ (new Date(item.created_at)).toLocaleString()}}</th>
          <td>{{ types[item.type] }}</td>
          <td v-if="item.working_out === 'ACTIVE'">
            <span style="cursor: pointer;" class="text-warning" v-on:click="updateStatus(item.id, 'DONE')" >Активно</span>
          </td>
          <td v-else-if="item.working_out === 'DONE'">
            <span class="text-success">Выполненно</span>
          </td>
          <td v-else>
            <span class="text-muted">Созданно</span>
          </td>
          <td style="max-width: 300px;">
            <p>{{item.note}}</p>
          </td>
          <td>
            <a target="_blank" :href="domain + '/storage/' + item.path_file">Просмотреть</a>
          </td>
          <td class="setting">
<!--            <p>Дата: 26.10.2022 18:15</p>-->
            <p>Исполнитель: {{item.employee.login}}</p>
            <p v-if="item.type === 11">Дата отработки: {{ item.working_out_limit_date }}</p>
            <p v-if="item.type === 11">Иполнитель отработки: {{ item.worging_employee.login }}</p>
<!--            <p>Дата окончания отработки: 26.10.2022</p>-->
          </td>
        </tr>
        </tbody>
      </table>
    </div>


    <br>
    <br>
    <br>
    <br>

    <div class="modal-show" v-if="!isHidden">
      <div class="modal fade in" id="add-cart-dialog" style="display: block; padding-right: 15px;">
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header">
              <button type="button" class="close" v-on:click="isHidden = !isHidden" data-dismiss="modal" aria-hidden="true">
                ×
              </button>
              <h4 class="modal-title" id="addCartTitleText">Создание заявления </h4>
            </div>
            <div class="modal-body">
              <form ref="form" id="form">
                <div class="form-group row">
                  <label class="col-sm-3 col-form-label">Событие</label>
                  <div class="col-sm-8">
                    <select class="form-control" aria-label="Default select example" v-model="form.type">
                      <option v-for="(type, index) in types" v-bind:key="index" :value="index">{{ type }}</option>
                    </select>
                  </div>
                </div>

                <div class="form-group row" v-if="form.type == 11">
                  <label class="col-sm-3 col-form-label">Дата отработки</label>
                  <div class="col-sm-8">
                    <input type="date" class="form-control" v-model="form.working_out_limit_date">
                  </div>
                </div>

                <div class="form-group row" v-if="form.type == 11">
                  <label class="col-sm-4 col-form-label">Исполнитель отработки</label>
                  <div class="col-sm-7">
                    <input type="date" class="form-control" v-model="form.working_out_to_employee">
                  </div>
                </div>

                <div class="form-group row">
                  <label class="col-sm-3 col-form-label">Коментарии</label>
                  <div class="col-sm-8">
                    <textarea class="form-control" v-model="form.note"></textarea>
                  </div>
                </div>

                <div class="form-group row">
                  <label class="col-sm-3 col-form-label">Фаил</label>
                  <div class="col-sm-8">
                    <input type="file" ref="file" class="form-control" @change="previewFiles">
                  </div>
                </div>

              </form>
            </div>
            <div class="modal-footer">
              <button type="button" class="btn btn-primary" v-on:click="save">Сохранить</button>
              <button type="button" class="btn btn-default" v-on:click="isHidden = !isHidden" data-dismiss="modal">Отмена</button>
            </div>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<script>

import axios from "axios";

export default {
  name: "Statement",
  components: {},

  props: {
    model: String,
    id: String,
  },
  data() {
    return {
      domain: 'http://127.0.0.1:8000',
      types: [],
      statements: [],
      filterStatements: [],
      isHidden: true,
      isFilter: false,
      form: {
        type: 1,
        path_file: {},
        note: '',
        working_out: "ACTIVE",
        working_out_limit_date: null,
        working_out_to_employee: 11,
      },
      errors: []
    }
  },
  methods:{
    async getTypes(){
      await axios.get(this.domain + '/api/statements/types').then(response => (this.types = response.data));
      console.log(this.types[1]);
    },
    async get(){
      await axios.get(this.domain + '/api/statements/' + this.model + '/' + this.id).then(response => (this.statements = response.data));
    },
    closeModal: function () {
      this.isHidden = false
    },
    previewFiles(event) {
      this.form.path_file = event.target.files[0]
      console.log(this.form.path_file)
    },
    save: function (){
      this.errors = [];
      let obj = this.form;

      if (obj.type != 11){
        delete obj.working_out;
        delete obj.working_out_limit_date;
        delete obj.working_out_to_employee;
      }

      let self = this;

      axios.post(this.domain + '/api/statements/student/11', obj, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }).then(response => {
        self.statements.unshift(response.data)
        self.form.type = 1;
        self.form.path_file = '';
        self.isHidden = !self.isHidden
      }).catch(error => {
        self.errors.push(error.response.data.message)
      });
    },
    updateStatus: function (id, status){
      let self = this;

      axios.post(this.domain + '/api/statements/updateWorking/' + id, {
        status: status,
      }, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }).then(response => {
        if (response.data){
          let obj = self.statements.find(o => o.id === id);
          obj.working_out = status;
        }
      }).catch(error => {
        self.errors.push(error.response.data.message)
      });
    },

    filterType: function (type){
      if (type == 0){
        this.isFilter = false
      }else {
        this.isFilter = true;
      }

      this.filterStatements = this.statements.filter(item => {
        return item.type == type;
      })
    },

    filterStatus: function (status){
      if (status == 0){
        this.isFilter = false
      }else {
        this.isFilter = true;
      }

      this.filterStatements = this.statements.filter(item => {
        return item.working_out === status;
      })
    },

    filterDate: function (date){
      if (date != '') {
        this.isFilter = true;
        this.filterStatements = this.statements.filter(item => {
          let created_at = (new Date(item.created_at)).toDateString();
          let d = (new Date(date)).toDateString();

          return created_at === d;
        })
      }else {
        this.filterStatements = []
        this.isFilter = false;
      }
    }
  },
  mounted() {
    this.getTypes();
    this.get();
  },
  computed: {
    filteredStatements(){
      if (this.filterStatements.length || this.isFilter){
        return this.filterStatements
      }

      return this.statements
    }
  },
}
</script>

<style scoped>
ul{
  padding: 0;
}

.setting{
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  max-width: 450px;
}

.modal {
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 1050;
  display: none;
  overflow: hidden;
  -webkit-overflow-scrolling: touch;
  outline: 0;
}

.modal-show{
  position: absolute;
  top: 0;
  left: 0;
  min-height: 100%;
  width: 100%;
  background: rgba(0, 0, 0, 0.39);
}
</style>