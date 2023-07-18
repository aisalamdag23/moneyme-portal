
<template>
    <b-form class="quotecalc" @submit="onSubmit">
        <div class="yourquote">
            <h1>Your Quote</h1>
            <div class="d-flex justify-content-between">
                <h4 class="mt-0">Your information</h4>
                <h5 class="mt-0 blue" @click="setRouteBack()">Edit</h5>
            </div>
            <div class="d-flex justify-content-between">
                <p class="mt-0">Name</p>
                <p class="mt-0">{{form.FirstName}} {{ form.LastName }}</p>
            </div>
            <div class="d-flex justify-content-between">
                <p class="mt-0">Mobile</p>
                <p class="mt-0">{{form.Mobile}}</p>
            </div>
            <div class="d-flex justify-content-between">
                <p class="mt-0">Email</p>
                <p class="mt-0">{{form.Email}}</p>
            </div>
            <div class="d-flex justify-content-between">
                <h4 class="mt-0 font-bold">Finance details</h4>
                <h5 class="mt-0 blue" @click="setRouteBack()">Edit</h5>
            </div>
            <div class="d-flex justify-content-between">
                <h5 class="mt-0">Finance amount</h5>
                <p class="mt-0 blue font-weight-bold">${{form.AmountRequired}}</p>
            </div>
            <div class="d-flex justify-content-between">
                <p class="mt-0">---------------------------------------------</p>
                <p class="mt-0">over {{ form.Term }} months</p>
            </div>
            <div class="d-flex justify-content-between">
                <h5 class="mt-0">Repayments from</h5>
                <p class="mt-0 blue font-weight-bold">${{form.Repayment}}</p>
            </div>
            <div class="d-flex justify-content-between">
                <p class="mt-0">---------------------------------------------</p>
                <p class="mt-0">Monthly</p>
            </div>
            <div  v-if="hasError" class="alert alert-danger" role="alert">
                {{ serverErrors }}
            </div>
            <b-button type="submit" variant="primary" class="mt-5 shadow-lg w-75" id="applyNow">Apply now</b-button>
            <div class="mt-2">
                <b-form-text class="font-small">Total repayments ${{ form.TotalRepayment }}, made up of an establishment fee of ${{ form.EstablishmentFee }}, interest of ${{ form.TotalInterest }}. The repayment amount is based on the variables selected, is subject to our assessment and suitability, and other important terms and conditions apply.</b-form-text>
            </div>
            <b-modal id="notifModal" class="modal-sm">
                <p class="my-4">Application successful! We'll let you know soon</p>
            </b-modal>
        </div>
    </b-form>
</template>

<style scoped>
    .yourquote {
        margin: auto;
        width: 50%;
        border: 3px solid #31787b;
        padding: 40px;
    }
    .blue {
        color: #31787b
    }

    #applyNow {
        background-color: #4bd133;
        border-color: #4bd133;
    }
</style>

<script>
  export default {
    data() {
      return {
        hasError: false,
        serverErrors: '',
        loanApp: null,
        form: {
            AmountRequired: '10000',
            Term: '12',
            Title: 'mr.',
            FirstName: null,
            LastName: null,
            DateOfBirth: null,
            Email: null,
            Mobile: null,
            Repayment: null,
            TotalInterest: null,
            EstablishmentFee: null,
            TotalRepayment: null
        }
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
                // Repayment
                this.form.Repayment = this.loanApp.Repayment
                // TotalInterest
                this.form.TotalInterest = this.loanApp.TotalInterest
                // EstablishmentFee
                this.form.EstablishmentFee = this.loanApp.EstablishmentFee
                // TotalRepayment
                this.form.TotalRepayment = String(parseFloat(this.loanApp.AmountRequired) + parseFloat(this.loanApp.TotalInterest) + parseFloat(this.loanApp.EstablishmentFee))
            })
            .catch(console.error);
        } 
        
    },
    methods: {
        setRouteBack() {
            return this.$router.push('/quotecalc?id='+this.$route.query.id)
        },
        onSubmit(e) {
            e.preventDefault();
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
                        this.$bvModal.show('notifModal');
                    }
                })
                .catch(console.error);
            } 
        }
    }
  }
</script>
