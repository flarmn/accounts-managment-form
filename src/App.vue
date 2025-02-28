
<template>
  <div class="accounts-form">
    <div class="accounts-form__header">
      <h2>Учетные записи</h2>
      <button class="accounts-form__header-button" @click="addAccount"> + </button>
    </div>

    <div class="info-message">
      <img src="@/assets/question_mark.png" class="info-message__sign" alt = "Info message icon">
      <p class="info-message__text">Для указания нескольких меток для одной пары логин/пароль используйте разделитель ;</p>
    </div>
    
    <div class="accounts-form__table-header">
      <div class="accounts-form__column-name-main-section">
        <label class="accounts-form__column-name">Метка</label>
        <label class="accounts-form__column-name">Тип записи</label>
      </div>
      
      <div class="accounts-form__column-name-login-section">
        <label class="accounts-form__column-name">Логин</label>
        <label class="accounts-form__column-name">Пароль</label>
      </div>
      
    </div>

    <div v-for="(account, index) in accounts" :key="account.id" class="account-card">
      <div class="account-card__cell-main">
        <input 
          :value="formatLabel(account.label)" 
          @input="updateLabel(index, $event.target.value)" 
          placeholder="Значение"
          maxlength="50"
        />

        <select v-model="account.type" @change="validateAndUpdate(index, 'type', account.type)">
          <option value="LDAP">LDAP</option>
          <option value="Локальная">Локальная</option>
        </select>
      </div>
      
      <div class="account-card__cell-login">
          <div class="account-card__cell-login-field">
            <input 
              v-model="account.login" 
              @blur="validateAndUpdate(index, 'login', account.login)" 
              placeholder="Значение"
              :class="{ 'invalid': errors[index]?.login }"
              maxlength="100"
            />
            <div class="error-message" v-if="errors[index]?.login">{{ errors[index]?.login }}</div>
          </div>
         
          <div class="account-card__cell-login-field" v-if="account.type === 'Локальная'">
            <input 
              v-if="account.type === 'Локальная'" 
              :type="passwordVisible[index] ? 'text' : 'password'"
              v-model="account.password" 
              @blur="validateAndUpdate(index, 'password', account.password)" 
              placeholder="Значение"
              :class="{ 'invalid': errors[index]?.password }"
              maxlength="100"
            />
            <div class="error-message" v-if="errors[index]?.password">{{ errors[index]?.password }}</div>
          </div>

          <button class="account-card__password-visibility-btn" @click.prevent="togglePasswordVisibility(index)" 
          v-if="account.type === 'Локальная' && account.password">
            {{ passwordVisible[index] ? '🙈' : '👁️' }}
          </button>
      </div>
      
      <button class="account-card__remove-btn" @click="removeAccount(index)"></button>
    </div>
</div>

</template>

<script>
import { computed } from 'vue';
import { useAccountsStore } from '@/stores/accounts';
import { ref } from 'vue';


export default {
  setup() {
  const store = useAccountsStore();
  const accounts = computed(() => store.accounts);

  const addAccount = () => store.addAccount();
  const removeAccount = (index) => store.removeAccount(index);

  const errors = ref({}); // Ошибки для полей

  const formatLabel = (label) => {
    if (!Array.isArray(label)) return ''; // Если label не массив, вернуть пустую строку
    return label.map(item => item.text).join('; ');
  };

  const updateLabel = (index, value) => {
    store.updateAccount(index, 'label', value.split(';').map(text => ({ text: text.trim() })));
  };

  const passwordVisible = ref({}); // Храним состояние для каждого аккаунта

  const togglePasswordVisibility = (index) => {
    passwordVisible.value[index] = !passwordVisible.value[index];
  };

  const validateAndUpdate = (index, field, value) => {
    if (!errors.value[index]) {
      errors.value[index] = {};
    }

    if (field === 'login') {
      if (!value.trim()) {
        errors.value[index].login = "Логин не может быть пустым";
      } else {
        delete errors.value[index].login;
      }
    }

    if (field === 'password') {
      const passwordRegex = /^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d]{8,}$/;

      if (!value.trim()) {
        errors.value[index].password = "Пароль не может быть пустым";
      } else if (!passwordRegex.test(value)) {
        errors.value[index].password = "Пароль должен содержать минимум 8 символов, включая буквы и цифры";
      } else {
        delete errors.value[index].password;
      }
    }

    // Если ошибок нет, обновляем store
    if (Object.keys(errors.value[index]).length === 0) {
      store.updateAccount(index, field, value);
    }
  };

  console.log(store.accounts);
  return { 
    accounts, 
    addAccount, 
    removeAccount, 
    formatLabel,
    updateLabel,
    validateAndUpdate,
    passwordVisible, 
    togglePasswordVisibility,
    errors
  };
}

};
</script>

<style>
.accounts-form {
  max-width: 70%;
  margin: auto;
  font-family: Arial, Helvetica, sans-serif;
}

.accounts-form__header {
  display: flex;
  align-items: center;
  line-height: 20px;
}

.accounts-form__header-button {
  font-size: 23px;
  margin-left: 10px;
  width: 35px;
  height: 35px;
  padding: 0;
  border: 1px solid #DDD;
  border-radius: 3px;
  background-color: white;
  color: grey;
  cursor: pointer;
}

.info-message {
  display: flex;
  height: 30px;
  background: lightgrey;
  align-items: center;
}

.info-message__sign {
  width: 20px;
  height: 20px;
  margin-left: 5px;
}

.info-message__text {
  margin-left: 10px;
  font-size: 14px;
}

.account-card,
.accounts-form__table-header {
  display: flex;
  margin-top: 25px;
  align-items: center;
  width: 100%;
  column-gap: 20px;
}

.accounts-form__column-name-main-section,
.account-card__cell-main,
.accounts-form__column-name-login-section,
.account-card__cell-login {
  display: flex;
  min-width: 55%;
  column-gap: 20px;
}

.accounts-form__column-name-main-section > .accounts-form__column-name,
.accounts-form__column-name-login-section > .accounts-form__column-name {
  display: flex;
  width: 50%;
  column-gap: 20px;
}

.accounts-form__column-name-login-section,
.account-card__cell-login {
  min-width: 40%;
}

.account-card__cell-login-field {
  display: flex;
  width: 50%;
  line-height: 25px;
  padding: 3px;
  flex-grow: 1;
}

input,
select {
  display: flex;
  width: 100%;
  flex-shrink: 1;
  flex-grow: 1;
  border-radius: 6px;
  line-height: 30px;
  border-width: 1px;
  padding-left: 10px;
}

button {
  margin-top: 5px;
  cursor: pointer;
}

.account-card__password-visibility-btn {
  position: absolute;
  margin-left: 26%;
  align-self: center;
  border-style: none;
  background-color: transparent;
  cursor: pointer;
}

.account-card__remove-btn {
  min-width: 25px;
  min-height: 25px;
  background: url('@/assets/delete.png') no-repeat center center;
  background-size: contain;
  border: none;
  cursor: pointer;
  padding: 0;
  margin: 0;
}

.error-message {
  position: absolute;
  width: 12%;
  font-size: 12px;
  color: red;
  margin-top: 35px;
  line-height: 12px;
  padding: 10px;
  background-color: rgba(221, 245, 39, 0.7);
  border-radius: 6px;
  z-index: 9000;
}

.invalid {
  border: 2px solid red;
  background-color: #ffe6e6;
}

.hidden {
  display: none;
}

/* mobile landscape */
@media (min-width: 280px) and (max-width: 767px) and (orientation: landscape) {
  .accounts-form {
    max-width: 80%;
  }

  .info-message {
    width: 100%;
    padding: 5px;
  }

  .account-card,
  .accounts-form__table-header {
    margin-top: 20px;
    column-gap: 10px;
  }

  .accounts-form__column-name-main-section,
  .account-card__cell-main,
  .accounts-form__column-name-login-section,
  .account-card__cell-login {
    display: flex;
    min-width: 50%;
    column-gap: 10px;
    flex-shrink: 1;
    flex-grow: 1;
  }

  .accounts-form__column-name-main-section > .accounts-form__column-name,
  .accounts-form__column-name-login-section > .accounts-form__column-name {
    display: flex;
    width: 50%;
    column-gap: 20px;
  }

  .account-card__password-visibility-btn {
    margin-left: 34%;
  }

  .error-message {
    width: 18%;
    font-size: 10px;
    margin-top: 32px;
    line-height: 12px;
    padding: 5px;
  }
}

/* mobile portrait */
@media (min-width: 280px) and (max-width: 767px) and (orientation: portrait) {
  .accounts-form {
    margin: 0;
    margin-left: 5%;
    max-width: 80%;
  }

  .accounts-form__header {
    width: 100%;
    justify-content: space-between;
    padding: 5px;
  }

  .accounts-form__table-header {
    display: flex;
  }

  .info-message {
    width: 100%;
    height: auto;
    xxline-height: auto;
    padding: 5px;
  }

  .account-card,
  .accounts-form__table-header {
    margin-top: 20px;
    column-gap: 10px;
  }

  .accounts-form__column-name-main-section,
  .account-card__cell-main,
  .accounts-form__column-name-login-section,
  .account-card__cell-login {
    display: flex;
    min-width: 50%;
    column-gap: 10px;
    flex-shrink: 1;
    flex-grow: 1;
  }

  .accounts-form__column-name-main-section > .accounts-form__column-name,
  .accounts-form__column-name-login-section > .accounts-form__column-name {
    display: flex;
    width: 50%;
    column-gap: 20px;
  }

  .account-card__cell-login-field {
    padding: unset;
  }

  .account-card__password-visibility-btn {
    margin-left: 30%;
  }

  .error-message {
    width: 18%;
    font-size: 10px;
    margin-top: 32px;
    line-height: 12px;
    padding: 5px;
  }
}

/* tablet */
@media (min-width: 768px) and (max-width: 1439px) {
  .accounts-form {
    max-width: 85%;
  }

  .info-message {
    width: 100%;
  }

  .account-card,
  .accounts-form__table-header {
    margin-top: 20px;
    column-gap: 10px;
  }

  .accounts-form__column-name-main-section,
  .account-card__cell-main,
  .accounts-form__column-name-login-section,
  .account-card__cell-login {
    display: flex;
    min-width: 50%;
    column-gap: 10px;
    flex-shrink: 1;
    flex-grow: 1;
  }

  .accounts-form__column-name-main-section > .accounts-form__column-name,
  .accounts-form__column-name-login-section > .accounts-form__column-name {
    display: flex;
    width: 50%;
    column-gap: 20px;
  }

  .account-card__password-visibility-btn {
    margin-left: 37%;
  }

  .error-message {
    width: 18%;
    font-size: 10px;
    margin-top: 32px;
    line-height: 12px;
    padding: 5px;
  }
}

</style>
