
<template>
  <div class="account-form">
    <div class = "accounts-form_header">
      <h2>Учетные записи</h2>
      <button class = "accounts-form_header-button" @click="addAccount"> + </button>
    </div>

    <div class = "info-message_container">
      <img src = "@/assets/question_mark.png" class = "info-message_sign">
      <p class = "info-message_text">Для указания нескольких меток для одной пары логин/пароль испольуйте разделитель ;</p>
    </div>
    
    <div class = "accounts-form_table-header">
      <div class = "accounts-form_column-name_main-section">
        <label class = "accounts-form_column-name">Метка</label>
        <label class = "accounts-form_column-name">Тип записи</label>
      </div>
      
      <div class = "accounts-form_column-name_login-section">
        <label class = "accounts-form_column-name">Логин</label>
        <label class = "accounts-form_column-name">Пароль</label>
      </div>
      
    </div>
    <div v-for="(account, index) in accounts" :key="account.id" class="account-card">
      <div class = "account-card_cell_main">
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
      
      <div class = "account-card_cell_login">
          <div class = "account-card_cell_login-field">
            <input 
            v-model="account.login" 
            @blur="validateAndUpdate(index, 'login', account.login)" 
            placeholder="Значение"
            :class="{ 'invalid': errors[index]?.login }"
            maxlength="100"
            />
            <div class = "error-message" v-if="errors[index]?.login !== 'undefined'" >{{ errors[index]?.login }}</div>
          </div>
         
        
       
          <div class = "account-card_cell_login-field" v-if="account.type === 'Локальная'" >
            <input 
            v-if="account.type === 'Локальная'" 
            :type="passwordVisible[index] ? 'text' : 'password'"
            v-model="account.password" 
            @blur="validateAndUpdate(index, 'password', account.password)" 
            placeholder="Значение"
            :class="{ 'invalid': errors[index]?.password }"
            maxlength="100"
            />
          <div class = "error-message" v-if="errors[index]?.password !== 'undefined'" >{{ errors[index]?.password }}</div>
        </div>

        <button class = "password-visibility-btn" @click.prevent="togglePasswordVisibility(index)" 
        v-if="account.type === 'Локальная' && account.password != null && account.password !== '' " >
          {{ passwordVisible[index] ? '🙈' : '👁️' }}
        </button>
      </div>
      
      <button class = "remove-btn" @click="removeAccount(index)"></button>
     
      
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
.account-form {
  max-width: 70%;
  margin: auto;
  font-family: Arial, Helvetica, sans-serif;
}

.accounts-form_header{
  display: flex;
  align-items: center;
  line-height: 20px;
}

.accounts-form_header-button{
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

.info-message_container{
  display: flex;
  height: 30px;
  background: lightgrey;
  align-items: center;
}

.info-message_sign{
width: 20px;
height: 20px;
margin-left: 5px;
}

.info-message_text{
  margin-left: 10px;
  font-size: 14px;
}

.account-card, .accounts-form_table-header{
  display: flex;
  margin-top: 25px;
  align-items: center;
  width: 100%;
  column-gap: 20px;
}

.accounts-form_column-name_main-section, .account-card_cell_main, 
.accounts-form_column-name_login-section, .account-card_cell_login{
  display: flex;
  min-width: 55%;
  column-gap: 20px;
}

.accounts-form_column-name_main-section > .accounts-form_column-name, 
.accounts-form_column-name_login-section > .accounts-form_column-name{
  display: flex;
  width: 50%;
  column-gap: 20px;
}

.accounts-form_column-name_login-section, .account-card_cell_login{
  min-width: 40%;
}

.account-card_cell_login-field{
  display: flex;
  width: 50%;
  line-height: 25px;
  padding: 3px;
  flex-grow: 1;
}

input, select {
  display: flex;
  width: 100%;
  flex-shrink: 1;
  flex-grow: 1;
  width: 100%;
  border-radius: 6px;
  line-height: 30px;
  border-width: 1px;
  padding-left: 10px;
}

button {
  margin-top: 5px;
  cursor: pointer;
}

.password-visibility-btn{
  position:absolute;
  margin-left: 26%;
  align-self: center;
  border-style:none;
  background-color: transparent;
  cursor: pointer;
}


.remove-btn{
  min-width: 25px;
  min-height: 25px;
  background: url('@/assets/delete.png') no-repeat center center;
  background-size: contain;
  border: none;
  cursor: pointer;
  padding:0;
  margin:0;
  cursor: pointer;
}

.error-message{
  position: absolute;
  width: 13%;
  font-size: 12px;
  color: red;
  margin-top: 32px;
  line-height: 12px;
  padding: 5px;
}

.invalid {
  border: 2px solid red;
  background-color: #ffe6e6;
}
</style>
