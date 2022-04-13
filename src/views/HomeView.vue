<template>
  <div class="home container">
    <button type="button" class="btn btn-primary mt-3" @click="signIn">
      Авторизоваться
    </button>
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
            language="ru"
            :options="{
              readOnly: !formLayout.active,
              i18n: formOptions.i18n,
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
      url: "https://open-newtemplate.isands.ru/api/",
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
      formOptions: {
        // readOnly: true,
        // noDefaults: true,
        i18n: {
          ru: {
            Name: "Имя",
            "Last name": "Фамилия",
            dict: "Тип заявителя",
            "Type to search": "Поиск...",
            "Last name is required": "Фамилия - это обязательное поле",
            "No results found": "Поиск не дал результатов",
            "is required": "обязательное поле",
            Number: "Число",
            Submit: "Подтвердить",
            Layout: "Расположение",
            "Drag and Drop a form component": "Переместите компонент сюда",
            "No Matches Found": "Ничего не найдено",
            "Text Field": "Текстовое поле",
            Email: "Электронная почта",
            "Text Area": "Текстовая область",
            "Phone Number": "Номер телефона",
            Checkbox: "Флажок",
            Select: "Выпадающий список",
            Radio: "Радио кнопка",
            Url: "Ссылка",
            "Data Map": "Ключ - Значение",
            "Data Grid": "Динамический список",
            "Edit Grid": "Сетка данных",
            Table: "Таблица",
            "Date / Time": "Дата / Время",
            Day: "День",
            Time: "Время",
            File: "Файл",
            Signature: "Подпись",
            Content: "Контент",
            Columns: "Столбцы",
            "Field Set": "Набор полей",
            Panel: "Панель",
            Tabs: "Вкладки",
            Well: "Лист",
            Label: "Название",
            "Please fix the following errors before submitting":
              "Пожалуйста исправьте ошибки перед теп как продолжить",
            "Email: Email must be a valid email.": "Не правильный e-mail",
            Placeholder: "Заполнитель",
            Description: "Описание",
            Tooltip: "Подсказка",
            "To add a tooltip to this field,enter text here.":
              "Введите подсказку здесь",
            "Input Mask": "Маска ввода",
            Hidden: "Скрытый",
            "Hide Label": "Скрыть название",
            Save: "Сохранить",
            Cancel: "Отмена",
            Remove: "Удалить",
            Preview: "Предварительный просмотр",
            Disabled: "Отключен",
            Validation: "Проверка",
            Data: "Данные",
            "Property Name": "Имя переменной",
            Display: "Отображение",
            Widget: "Тип компонента",
            required: "обязательно для заполнения",
            pattern: "не соответствует маске!",
            error: "Пожалуйста исправьте ошибки прежде чем продолжить.",
            submitError:
              "Пожалуйста исправьте все ошибки прежде чем продолжить.",
            invalid_regex: "не соответствует маске!",
            mask: "{{field}} не соответствует маске.",
            valueIsNotAvailable: "неправильное значение.",
            Edit: "Редактировать",
            "Label Position": "Расположение",
            "Label Width": "Ширина",
            "Label Margin": "Отступ",
            Prefix: "Прификс",
            Suffix: "Суффикс",
            "Custom CSS Class": "CSS класс",
            "Show Word Counter": "Показать счетчик слов",
            "Show Character Counter": "Показать счетчик символов",
            "Hide Input": "Скрыть ввод",
            "Initial Focus": "Начальный фокус",
            "Allow Spellcheck": "Проверка орфографии",
            "Modal Edit": "Показать во всплывающем окне",
            "Tab Index": "Индекс вкладки",
            Autocomplete: "Автозавершение",
            month: "Месяц",
            day: "День",
            year: "Год",
            january: "Январь",
            february: "Февраль",
            march: "Март",
            april: "Апрель",
            may: "Май",
            june: "Июнь",
            july: "Июль",
            august: "Август",
            september: "Сентябрь",
            october: "Октябрь",
            november: "Ноябрь",
            december: "Декабрь",
            next: "Далее",
            previous: "Назад",
            cancel: "Отмена",
            submit: "Отправить",
          },
        },
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
        userId: 632001,
        appId: this.formLayout.id,
        data: JSON.stringify(this.formLayout.data),
      };
      axios
        .post(this.url + "app/action-invoke", request)
        .then((response) => {
          console.log("Ответ на экшн");
          console.log(response);
          if (response.data.responseObject) {
            let fileApp = JSON.parse(response.data.responseObject);
            console.log("Объект файла");
            console.log(fileApp);
            let link = document.createElement("a");
            link.setAttribute("download", fileApp.fileName);
            link.setAttribute(
              "href",
              "data:application/octet-stream;base64," + fileApp.fileData
            );
            link.click();
          } else {
            if (isBackAction) {
              this.$router.push("/registry");
            } else {
              this.getNextForm(response);
            }
          }
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

    signIn() {
      let bodyFormData = new FormData();
      bodyFormData.append("login", "mikhail");
      bodyFormData.append("password", "12345");
      // let signForm = document.getElementById(signForm);
      console.log(bodyFormData);
      // signForm.submit();
      axios.post(this.url + "auth/local-login", {
        login: "mikhail",
        password: "12345",
      });
    },
  },
};
</script>

<style lang="scss">
.btn {
  word-wrap: break-word;
}
</style>
