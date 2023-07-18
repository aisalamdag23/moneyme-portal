
<template>
    <b-form class="quotecalc" @submit="onSubmit">
        <h1>Quote Calculator</h1>
        <div class="qc-amount">
            <b-form-input v-model="form.AmountRequired" type="range" :min="qcAmountMin" :max="qcAmountMax" class="slider"></b-form-input>
            <output class="bubble">${{ form.AmountRequired }}</output>
            <div class="d-flex justify-content-between">
                <b-form-text class="mt-0">$2,100</b-form-text>
                <b-form-text class="mt-0">How much do you need</b-form-text>
                <b-form-text class="mt-0">$15,000</b-form-text>
            </div>
        </div>
        <div class="qc-term mt-5">
            <b-form-input v-model="form.Term" type="range" min="1" max="36" class="slider"></b-form-input>
            <output class="bubble">{{ form.Term }} mos</output>
            <div class="d-flex justify-content-between">
                <b-form-text class="mt-0">1 mo</b-form-text>
                <b-form-text class="mt-0">How long will you pay for it</b-form-text>
                <b-form-text class="mt-0">36 mos</b-form-text>
            </div>
        </div>
        <div class="qc-name mt-5">
            <b-input-group>
                <b-form-select class="border-0 border-bottom mr-2" v-model="form.Title" :options="nameTitleOpts"></b-form-select>
                <span class="ig-spacer"></span>
                <b-form-input aria-label="First name" class="border-0 border-bottom" placeholder="First name" v-model="form.FirstName"></b-form-input>
                <span class="ig-spacer"></span>
                <b-form-input aria-label="Last name" class="border-0 border-bottom" placeholder="Last name" v-model="form.LastName"></b-form-input>
            </b-input-group>
        </div>
        <div class="qc-dob mt-5">
            <b-input-group>
                <label for="dobInput" class="mt-1">Date of Birth</label>
                <span class="ig-spacer"></span>
                <VueDatePicker
                    v-model="form.DateOfBirth"
                    format="YYYY-MM-DD"
                    id="dobInput"
                />
            </b-input-group>
        </div>
        <div class="qc-email-mobile mt-5">
            <b-input-group>
                <b-form-input aria-label="Your Email" class="border-0 border-bottom" placeholder="Your Email" v-model="form.Email"></b-form-input>
                <span class="ig-spacer"></span>
                <b-form-input aria-label="Mobile number" class="border-0 border-bottom" placeholder="Mobile number" v-model="form.Mobile"></b-form-input>
            </b-input-group>
        </div>
        <div class="mt-5">
            <div  v-if="hasError" class="alert alert-danger" role="alert">
                {{ serverErrors }}
            </div>
            <b-button type="submit" variant="primary" class="mt-2 shadow-lg" id="calcQuoteSubmit">Calculate quote</b-button>
        </div>
        <div class="mt-2">
            <b-form-text class="font-small">Quote does not affect your credit score</b-form-text>
        </div>
    </b-form>
</template>

<style scoped>
    .quotecalc {
        margin: auto;
        width: 50%;
        border: 3px solid #31787b;
        padding: 40px;
    }
    .slider {
        width: 100%;
        accent-color: #03d1cf;
    }
    .ig-spacer {
        width: 10px;
    }

    .bubble {
        background-color: #03d1cf;
        padding: 2px 15px;
    }

    #calcQuoteSubmit {
        background-color: #4bd133;
        border-color: #4bd133;
    }
</style>

<script>
import { VueDatePicker } from '@mathieustan/vue-datepicker';
import '@mathieustan/vue-datepicker/dist/vue-datepicker.min.css';

  export default {
    components : {
        VueDatePicker,
    },
    data() {
        const now = new Date()
        const today = new Date(now.getFullYear(), now.getMonth(), now.getDate())
        // 15th two months prior
        const minDate = new Date(today)
        minDate.setMonth(minDate.getFullYear() - 50)
        minDate.setDate(15)
        // 15th in two months
        const maxDate = new Date(today)
        maxDate.setMonth(maxDate.getFullYear() - 18)
        maxDate.setDate(15)

      return {
        loanApp: null,
        hasError: false,
        form: {
            AmountRequired: '10000',
            Term: '12',
            Title: 'mr.',
            FirstName: null,
            LastName: null,
            DateOfBirth: null,
            Email: null,
            Mobile: null,
        },
        nameTitleOpts: [
          { value: 'mr.', text: 'Mr.' },
          { value: 'ms.', text: 'Ms.' },
          { value: 'mrs.', text: 'Mrs.' },
        ],
        qcAmountMin: '2100',
        qcAmountMax: '15000',
        qcDOBMin: minDate,
        qcDOBMax: maxDate,
        serverErrors: '',
        formatted: '',
        selected: ''
      }
    },
    created() {
        if (this.$route.query.id !== undefined) {
            fetch(process.env.VUE_APP_API + '/loan-app/'+this.$route.query.id , {
                method: 'get'
            })
            .then(async (data) => {
                this.loanApp = await data.json();
                // amount 
                console.log(this.loanApp)
                this.form.AmountRequired = this.loanApp.AmountRequired
                // term
                this.form.Term = this.loanApp.Term
                // title
                this.form.Title = this.loanApp.Title.toLowerCase()
                // FirstName
                this.form.FirstName = this.loanApp.FirstName
                // LastName
                this.form.LastName = this.loanApp.LastName
                // bod
                this.form.DateOfBirth = this.loanApp.DateOfBirth
                // Email
                this.form.Email = this.loanApp.Email
                // Mobile
                this.form.Mobile = this.loanApp.Mobile
            })
            .catch(console.error);
        } 
        
    },
    methods: {
        onSubmit(e) {
            e.preventDefault()
            if (this.$route.query.id !== undefined) {
                fetch(process.env.VUE_APP_API + '/loan-app/'+this.$route.query.id , {
                    method: 'put',
                    headers: {'Content-type': 'application/json'},
                    body: JSON.stringify(this.form)
                })
                .then(async (data) => {
                    const respBody = await data.json();
                    if (data.status !== 200) {
                        this.serverErrors = respBody.message;
                        this.hasError = true;
                    } else {
                        this.$router.push('/yourquote?id='+this.$route.query.id)
                    }
                })
                .catch(console.error);
            } 
        },
    }
  }
</script>
