<template>

    <div style="display: inline-block" class="balanceShowContent">

        <loading-spinner class="fontColor spinnerBalance" v-if="!this.loaded" />
        <div class="show-balance-span" v-if="this.loaded" >
            {{ this.formatMoneyNumber( (this.computePrice +  (this.showPoolReward === true ? this.computePoolReward : 0 )) ,2)}}
        </div>

    </div>

</template>

<script>

    import LoadingSpinner from "components/UI/elements/Loading-Spinner.vue"
    import BrowserHelpers from "helpers/Browser.helpers"

    export default{

        components:{
            LoadingSpinner,
        },

        props: ['address', 'currency', 'showPoolReward'],

        data(){
          return {

              balances: {},
              subscription: null,
              loaded: WebDollar.Blockchain.loaded||false,

              minerPoolTotalReward: 0,
              minerPoolConfirmedReward: 0,

            }
        },

        computed:{

            computePrice(){

                if (this.balances === null || this.balances === undefined || !this.balances.hasOwnProperty(this.currency)) return 0;

                return (this.balances[this.currency]);
            },

            computePoolReward(){

                return this.minerPoolTotalReward + this.minerPoolConfirmedReward;

            }

        },

        mounted(){

            if (typeof window === "undefined") return;

            this.currency = this.currency || '0x01';

            let address = this.address;
            if (typeof this.address === "object" && typeof this.address.hasOwnProperty("address") ) { //it is an address object
                address = this.address.address;
            }

            WebDollar.StatusEvents.emitter.on("blockchain/status", (data)=>{

                if (data.message === "Blockchain Ready to Mine")
                    this.loaded = true;

            });

            let data = WebDollar.Blockchain.Balances.subscribeBalancesChanges(address, (data)=>{
                this.balances = data.balances;
            });

            if (data !== null && data.result) {
                this.subscription = data.subscription;
                this.balances = data.balances;
            }



            //pool reward

            if (WebDollar.Blockchain.MinerPoolManagement !== undefined) {
                this.minerPoolTotalReward = WebDollar.Blockchain.MinerPoolManagement.minerPoolReward.totalReward;
                this.minerPoolConfirmedReward = WebDollar.Blockchain.MinerPoolManagement.minerPoolReward.confirmedReward;
            }

            WebDollar.StatusEvents.on("miner-pool/total-reward", (data)=>{
                this.minerPoolTotalReward = data.totalReward;
            });

            WebDollar.StatusEvents.on("miner-pool/confirmed-reward", (data)=>{
                this.minerPoolConfirmedReward = data.confirmedReward;
            });

        },

        watch: {

            address: function (newVal, oldVal) { // watch it

                WebDollar.Blockchain.Balances.unsusbribeBalancesChanges(this.subscription);

                let address = newVal;
                if (typeof newVal === "object" && typeof newVal.hasOwnProperty("address") ) { //it is an address object
                    address = newVal.address;
                }

                console.log("address!!!!!!", address);

                let data = WebDollar.Blockchain.Balances.subscribeBalancesChanges(address, (data)=>{
                    console.log("balance changed");
                    this.balances = data.balances;
                });

                if (data !== null && data.result) {
                    this.subscription = data.subscription;
                    this.balances = data.balances;
                }

            },

            currency: function (newVal, oldVal) { // watch it

            },

            showPoolReward: function (newVal, oldVal) { // watch it

            }


        },

        methods:{

            formatMoneyNumber: BrowserHelpers.formatMoneyNumber,

        }

    }

</script>

<style>

    .balanceContent .fontColor{
        display: block;
    }

    .balanceContent .miningAddress{
        margin-top: 30px;
    }

    .balanceContent  .show-balance-span{
        font-size:20px;
    }

    .show-balance-span{
        display: inline-block;
        margin-right: 4px;
        color: #fec02c;
        vertical-align: top;
        margin-top: 0;
        text-align: center;
    }

    .balanceContent .show-balance-span{
        margin-top: 0;
        font-size: 14px;
    }

    .balanceShowContent{
        box-sizing: border-box;
    }

    @media only screen and (max-width : 831px) {

        .show-balance-span{
            margin-left: 5px;
            line-height: 48px;
        }

    }

    @media only screen and (max-width : 600px) {

        .walletAddress b{
            margin-top: 0!important;
        }

        .addressIdentityBox .show-balance-span{
            line-height: 40px;
            font-size: 14px;
        }

        .addressIdentityBox .show-balance-span svg{

            margin-top: 15px;
            display: block;

        }

    }
</style>