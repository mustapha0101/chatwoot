<template>
  <form
    class="flex flex-1 flex-col p-6 overflow-y-auto"
    @submit.prevent="onSubmit"
  >
    <div v-if="options.preChatMessage" class="text-black-800 text-sm leading-5">
      {{ options.preChatMessage }}
    </div>
    <form-input
      v-if="options.requireEmail"
      v-model="fullName"
      class="mt-5"
      :label="$t('PRE_CHAT_FORM.FIELDS.FULL_NAME.LABEL')"
      :placeholder="$t('PRE_CHAT_FORM.FIELDS.FULL_NAME.PLACEHOLDER')"
      type="text"
      :error="
        $v.fullName.$error ? $t('PRE_CHAT_FORM.FIELDS.FULL_NAME.ERROR') : ''
      "
    />
    <form-input
      v-if="options.requireEmail"
      v-model="emailAddress"
      class="mt-5"
      :label="$t('PRE_CHAT_FORM.FIELDS.EMAIL_ADDRESS.LABEL')"
      :placeholder="$t('PRE_CHAT_FORM.FIELDS.EMAIL_ADDRESS.PLACEHOLDER')"
      type="email"
      :error="
        $v.emailAddress.$error
          ? $t('PRE_CHAT_FORM.FIELDS.EMAIL_ADDRESS.ERROR')
          : ''
      "
    />
    <form-text-area
      v-model="message"
      class="my-5"
      :label="$t('PRE_CHAT_FORM.FIELDS.MESSAGE.LABEL')"
      :placeholder="$t('PRE_CHAT_FORM.FIELDS.MESSAGE.PLACEHOLDER')"
      :error="$v.message.$error ? $t('PRE_CHAT_FORM.FIELDS.MESSAGE.ERROR') : ''"
    />
    <woot-button
      class="font-medium"
      block
      :bg-color="widgetColor"
      :text-color="textColor"
      :disabled="isCreating"
    >
      <spinner v-if="isCreating" class="p-0" />
      {{ $t('START_CONVERSATION') }}
    </woot-button>
  </form>
</template>

<script>
import WootButton from 'shared/components/Button';
import FormInput from '../Form/Input';
import FormTextArea from '../Form/TextArea';
import Spinner from 'shared/components/Spinner';
import { mapGetters } from 'vuex';
import { getContrastingTextColor } from 'shared/helpers/ColorHelper';
import { required, minLength, email } from 'vuelidate/lib/validators';
import { IFrameHelper } from '../../../sdk/IFrameHelper';
export default {
  components: {
    FormInput,
    FormTextArea,
    WootButton,
    Spinner,
  },
  props: {
    options: {
      type: Object,
      default: () => ({}),
    },
  },
   mounted() {
    const { locale } = window.chatwootWebChannel;
   
  },
  validations() {
    const identityValidations = {
      fullName: {
        required,
      },
      emailAddress: {
        required,
        email,
      },
    };

    const messageValidation = {
      message: {
       // required,
        minLength: minLength(1),
      },
    };
    if (this.options.requireEmail) {
      return {
        ...identityValidations,
        ...messageValidation,
      };
    }
    return messageValidation;
  },
  data() {
    return {
      fullName: '',
      emailAddress: '',
      message: '',
    };
  },
  computed: {
    ...mapGetters({
      widgetColor: 'appConfig/getWidgetColor',
      isCreating: 'conversation/getIsCreating',
    }),
    textColor() {
      return getContrastingTextColor(this.widgetColor);
    },
  },
  methods: {
    onSubmit() {
      this.$v.$touch();
      if (this.$v.$invalid) {
        return;
      }
      // const { locale } = IFrameHelper.getLocale();
    /*   var tmpMessage; 
       if(!this.message){
         if(locale === "en"){

          tmpMessage = locale;
         }else {

          tmpMessage = locale;
         }
         
       }else {
         tmpMessage= this.message;

       }*/
      this.$store.dispatch('conversation/createConversation', {
        fullName: this.fullName,
        emailAddress: this.emailAddress,
        message:this.message,
        private:true,
      });
    },
  },
};
</script>
