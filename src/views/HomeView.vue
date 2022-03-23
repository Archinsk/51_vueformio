<template>
  <div class="home container">
    <div class="card my-3">
      <div class="card-body d-flex justify-content-between align-items-center">
        <b>{{ measure.id }}. {{ measure.name }}</b>
        <button class="btn btn-outline-primary" @click="getStartForm">
          Подать заявку
        </button>
      </div>
    </div>
    <!--    <Form src="https://dkpwtpfnbsdzufs.form.io/fioformwithdata" />-->
    <template v-if="isResponse">
      <div v-if="isAlertVisible" class="alert alert-success" role="alert">
        {{ successComment }}
      </div>
      <h4 class="text-center">{{ formLayout.form.name }}</h4>
      <div class="row">
        <div class="col-10">
          <Form
            :form="formLayout.form.scheme"
            :submission="formLayout"
            :options="{
              readOnly: !formLayout.active,
            }"
            ref="vueForm"
          />
        </div>
        <div
          v-if="formLayout.form.actions && formLayout.form.actions.length > 0"
          class="col-2"
        >
          <template v-for="action of formLayout.form.actions">
            <template v-if="formLayout.active || action.alwaysActive">
              <button
                v-if="action.backAction"
                :key="action.id"
                type="button"
                class="btn btn-block btn-primary mb-2"
                @click="invokeAction(action.id, true)"
              >
                {{ action.name }}
              </button>
              <button
                v-else
                :key="action.id"
                type="button"
                class="btn btn-block btn-primary mb-2"
                @click="invokeAction(action.id)"
              >
                {{ action.name }}
              </button>
            </template>
          </template>
        </div>
      </div>
    </template>
    <template v-else-if="isLoading">
      <div class="card my-3">
        <div class="card-body text-center">
          <div class="spinner-border text-primary" role="status">
            <span class="sr-only">Loading...</span>
          </div>
          <div>{{ loadingComment }}</div>
        </div>
      </div>
    </template>
  </div>
</template>

<script>
import { Form } from "vue-formio";
import axios from "axios";

export default {
  name: "HomeView",
  components: {
    Form,
  },

  data() {
    return {
      url: "http://192.168.18.171:8080/api/",
      measure: {
        id: 67,
        name: "OpenPartTestModel",
      },
      // measure: {
      //   id: 45,
      //   name: "Для открытого контура",
      // },
      isLoading: false,
      loadingComment: "Загрузка формы заявления",
      successComment: "Форма заявления успешно загружена!",
      isResponse: false,
      isAlertVisible: false,
      formLayout: {
        active: false,
        data: {},
        form: {
          actions: [],
          id: 0,
          modelId: 0,
          scheme: {},
        },
        id: 0,
        orderId: "",
        status: "",
      },
      isValidFormData: false,
      isFirstLoad: true,
    };
  },

  methods: {
    // Стартовая форма заявления
    getStartForm() {
      this.isResponse = false;
      this.isLoading = true;
      this.loadingComment = "Загрузка формы заявления";
      setTimeout(this.getForm, 1000);
    },
    getForm() {
      axios
        .get(this.url + "serv/get-appData?id=" + this.measure.id)
        .then((response) => {
          console.log("Стартовая форма");
          console.log(response);
          const newForm = response.data.applicationDTO;
          newForm.data = JSON.parse(newForm.data);
          newForm.form.scheme = JSON.parse(newForm.form.scheme);
          this.formLayout = newForm;
        })
        .then(() => {
          this.isResponse = true;
          this.isLoading = false;
          this.isAlertVisible = true;
          setTimeout(this.hideAlert, 3000);
        });
    },

    hideAlert() {
      this.isAlertVisible = false;
    },

    validateForm() {
      return this.$refs.vueForm.formio.checkValidity(
        this.$refs.vueForm.formio.submission.data
      );
    },

    invokeAction(actionId, isBackAction = false) {
      console.log("Выполнение действия");
      this.isFirstLoad = false;
      this.isValidFormData = this.validateForm();
      console.log("Валидность формы:" + this.isValidFormData);
      if (this.isValidFormData) {
        this.loadingComment = "Отправка данных заявления";
        this.isResponse = false;
        this.isLoading = true;
        setTimeout(this.invoke, 1000, actionId, isBackAction);
      } else {
        this.$refs.vueForm.formio.submit();
      }
    },
    invoke(actionId, isBackAction = false) {
      const request = {
        actionId: actionId,
        userId: 13,
        appId: this.formLayout.id,
        data: JSON.stringify(this.formLayout.data),
      };
      axios
        .post(this.url + "app/action-invoke", request)
        .then((response) => {
          console.log("Ответ на экшн");
          console.log(response);
          // if (response.data.applicationDTO.form) {
          if (isBackAction) {
            this.$router.push("/registry");
          } else {
            this.getNextForm(response);
          }
          // } else {
          //   let responseData = JSON.parse(response.data.applicationDTO.data);
          //   console.log("Распарсенные данные");
          //   console.log(responseData);
          //   console.log("id запрашиваемого заявления");
          //   console.log(responseData.params_handler_application);
          //   this.getAppExtData(responseData.params_handler_application);
          // }
        })
        .then(() => {
          this.isResponse = true;
          this.isLoading = false;
          this.isAlertVisible = true;
          this.isFirstLoad = true;
          setTimeout(this.hideAlert, 3000);
        });
    },

    // Переход к следующей форме (стандартное дейтвие)
    getNextForm(response) {
      console.log("Следующая форма");
      console.log(response);
      let nextForm = response.data.applicationDTO;
      nextForm.data = JSON.parse(nextForm.data);
      nextForm.form.scheme = JSON.parse(nextForm.form.scheme);
      this.formLayout = nextForm;
      this.successComment = "Заявление отправлено!";
    },
    // Запрос заявления id закрытого контура
    // getAppExtData(id) {
    //   console.log("Сохранение заявления");
    //   axios
    //     .get(this.url + "app/get-appExtData?extId=" + id)
    //     .then((response) => {
    //       console.log("Ответ на запрос заявления");
    //       console.log(response);
    //       const newForm = response.data;
    //       newForm.data = JSON.parse(newForm.data);
    //       newForm.form.scheme = JSON.parse(newForm.form.scheme);
    //       this.formLayout = newForm;
    //       this.successComment = "Заявление сохранено!";
    //     });
    // },
  },
};
</script>

<style lang="scss">
.btn {
  word-wrap: break-word;
}
</style>
