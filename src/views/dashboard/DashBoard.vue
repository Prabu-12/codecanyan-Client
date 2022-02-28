<template>
    <v-container class="px-0">
        <v-row v-if="itemsQuiz.length==0 && roleName=='Student'" justify="center" class="font-weight-bold">No Assessment Available!</v-row>
        <v-row v-else-if="itemsQuiz.length>0 && roleName=='Student'" justify="center" class="font-weight-bold">Available Assessments</v-row>
        <v-row justify="center" v-if="roleName=='Student'">
            <v-col cols="12" sm="6" md="4" v-for="item in itemsQuiz" :key="item.quizTopicId">
                <v-card color="grey lighten-3" shaped elevation="6">
                    <v-card-text class="pb-0">
                        <p class="title text--primary">{{item.quizTitle}}</p>
                    </v-card-text>
                    <v-list class="pa-0">
                        <v-list-item class="py-0">
                            <v-list-item-icon><v-icon>timer</v-icon></v-list-item-icon>
                            <v-list-item-title>Time</v-list-item-title>
                            <v-list-item-title>{{item.quizTime==0?'':item.quizTime+' minutes'}}</v-list-item-title>
                        </v-list-item>
                        <v-list-item class="py-0">
                            <v-list-item-icon><v-icon>grade</v-icon></v-list-item-icon>
                            <v-list-item-title>Marks</v-list-item-title>
                            <v-list-item-title>{{item.quizTotalMarks==0?'':item.quizTotalMarks}}</v-list-item-title>
                        </v-list-item>
                        <v-list-item class="py-0">
                            <v-list-item-icon><v-icon>help</v-icon></v-list-item-icon>
                            <v-list-item-title>Questions</v-list-item-title>
                            <v-list-item-title>{{item.questionsCount}}</v-list-item-title>
                        </v-list-item>
                    </v-list>
                    <v-divider></v-divider>
                    <v-card-actions>
                        <form v-bind:action="stripePostUrl" method="POST" v-if="item.quizPrice>0 && item.paymentCount==0">
                            <input type="hidden" name="quizTopicId" v-bind:value="item.quizTopicId">
                            <v-btn type="submit" class="white--text" color="grey darken-2" @click="saveQuizTopicId(item)">Pay {{getCurrency}}{{item.quizPrice}}</v-btn>
                        </form>
                        <v-btn v-else class="white--text" color="grey darken-2" @click="startQuiz(item)">Start</v-btn>
                    </v-card-actions>                   
                </v-card>
            </v-col>
        </v-row>
        <v-row v-if="roleName=='Admin'">
            <v-col class="pl-2" cols="12" sm="6" md="3">
                <v-card :to="{name:'Users'}" class="grey lighten-1">
                    <v-card-title>
                        <v-icon x-large left>mdi-account-multiple</v-icon>       
                    </v-card-title>
                    <v-card-text>Candidates <span class="black--text">{{totalStudents}}</span></v-card-text>
                </v-card>
            </v-col>
            <v-col class="pl-2" cols="12" sm="6" md="3">
                <v-card :to="{name:'QuizTopics'}" class="grey lighten-2">
                    <v-card-title>
                        <v-icon x-large left>emoji_objects</v-icon>       
                    </v-card-title>
                    <v-card-text>Assessments <span class="black--text">{{totalQuizes}}</span></v-card-text>
                </v-card>
            </v-col>
            <v-col class="pl-2" cols="12" sm="6" md="3">
                <v-card :to="{name:'QuizTopics'}" class="grey lighten-3">
                    <v-card-title>
                        <v-icon x-large left>emoji_objects</v-icon>       
                    </v-card-title>
                    <v-card-text>Live Assessments <span class="black--text">{{liveQuizes}}</span></v-card-text>
                </v-card>
            </v-col>
            <v-col class="pl-2" cols="12" sm="6" md="3">
                <v-card :to="{name:'Quizes'}" class="grey lighten-4">
                    <v-card-title>
                        <v-icon x-large left>help_center</v-icon>       
                    </v-card-title>
                    <v-card-text>Questions <span class="black--text">{{totalQuestions}}</span></v-card-text>
                </v-card>
            </v-col>
        </v-row>
        <v-row>
            <v-col cols="12">
                <line-chart v-if="loadQuiz" :chartdata="quizCountdata" :options="chartOptions"/>
            </v-col>
        </v-row>
        <v-row>
            <v-col cols="12">
                <bar-chart v-if="loadUser" :chartdata="quizUserCountdata" :options="barChartOptions"/>
            </v-col>
        </v-row>
        <v-row>
            <v-col cols="12" md="4">
                <pie-chart v-if="loadMonth" :chartdata="monthdata" :options="chartOptions"/>
            </v-col>
            <v-col cols="12" md="4">
                <pie-chart v-if="loadBrowser" :chartdata="browserdata" :options="chartOptions"/>
            </v-col>
            <v-col cols="12" md="4">
                <pie-chart v-if="loadPlatform" :chartdata="platformdata" :options="chartOptions"/>
            </v-col>
        </v-row>
        <v-row>
            <v-col cols="12">
                <line-chart v-if="loadDate" :chartdata="datedata" :options="chartOptions"/>
            </v-col>
        </v-row>
    </v-container>
</template>

<script>
import config from '../../config'
import LineChart from '../../assets/chart/lineChart'
import BarChart from '../../assets/chart/barChart'
import PieChart from '../../assets/chart/pieChart'

export default {
    name:'Dashboard',
    components:{
        LineChart,
        BarChart,
        PieChart
    },
    data(){
        return{           
            itemsQuiz:[],  
            userInfo:{},
            totalStudents:'',
            totalQuizes:'',
            liveQuizes:'',
            totalQuestions:'',
            loadQuiz:false,
            quizCountdata:null,
            chartOptions:null,
            loadUser:false,
            quizUserCountdata:null,
            barChartOptions:null,
            loadMonth:false,
            monthdata:null,
            loadBrowser:false,
            browserdata:null,
            loadPlatform:false,
            platformdata:null,
            loadDate:false,
            datedata:null,
            userId:null
        }
    },
    methods:{
        initializeStudent(){
            this.$store.dispatch('dashboard/applyLoading')
            this.$store.dispatch('dashboard/fetchLiveQuizes',this.userInfo.email)
            .then((response)=>{
                this.$store.dispatch('dashboard/cancelLoading')
                this.itemsQuiz=response.data
            })
            .catch((err)=>{
                console.log(err)
            })
        },
        initializeAdmin(){
            this.$store.dispatch('dashboard/applyLoading')
            this.$store.dispatch('dashboard/fetchSummary')
            .then((response)=>{
                this.$store.dispatch('dashboard/cancelLoading')
                if(response.status==200){
                    this.totalStudents=response.data.totalStudents
                    this.totalQuizes=response.data.totalQuizes
                    this.liveQuizes=response.data.liveQuizes
                    this.totalQuestions=response.data.totalQuestions
                }
            })
            .catch((err)=>{
                console.log(err)
            })
        },
        startQuiz(item){
            this.$store.dispatch('dashboard/applyLoading')
            const objResponseInitial={
                userId:parseInt(localStorage.getItem('loggedUserId')),
                email:this.userInfo.email,
                quizTopicId:item.quizTopicId,
                quizTitle:item.quizTitle,
                quizMark:item.quizTotalMarks,
                quizPassMarks:item.quizPassMarks,
                quizTime:item.quizTime,
                addedBy:parseInt(localStorage.getItem('loggedUserId'))
            }
            this.$store.dispatch('dashboard/createInitialResponse',objResponseInitial)
            .then((response)=>{
                this.$store.dispatch('dashboard/cancelLoading')
                if(response.status==200){
                    this.$store.dispatch('dashboard/startQuiz',item)
                    this.$store.dispatch('dashboard/saveQuizLiveTime',item.quizTime)
                    this.$router.push({name:'StartQuiz'})
                }
            })
        },
        saveQuizTopicId(item){
            localStorage.setItem('quizTopicId', item.quizTopicId)
        },
        makePayment(){
            var url = new URL(window.location.href);
            var sessionId = url.searchParams.get('session_id');
            if(sessionId){
                var uniqueId=sessionId.replace(/{|}/g,'')
                fetch(config.hostname+'/checkout-session?sessionId='+uniqueId)
                .then(response => response.json())
                    .then(data=>{
                            if(data.id==uniqueId && data.paymentStatus=='paid'){                           
                                const obj={
                                    quizTopicId:parseInt(localStorage.getItem('quizTopicId')),
                                    email:this.userInfo.email,
                                    currency:this.$store.getters['settings/currency'],
                                    sessionId:uniqueId,
                                    addedBy:this.userInfo.userId,
                                }
                                this.$store.dispatch('quiz/createPayment',obj)
                                .then(response=>{
                                    if(response.status==200){
                                        location.replace(window.location.origin+'/dashboard')                                       
                                    }else if(response.status==202){
                                        this.$root.$emit('message_from_parent',response.data.responseMsg)
                                    }
                                })
                                .catch(err=>{
                                    console.log(err)
                                })
                            }
                        }
                    )
                .catch(function (err) {
                    console.log('Error when fetching Checkout session', err);
                });
            }
        }
    },
    mounted(){
        this.loadQuiz=false
        this.loadUser=false
        this.loadMonth=false
        this.loadBrowser=false
        this.loadPlatform=false
        this.loadDate=false
        this.userId=parseInt(localStorage.getItem('loggedUserId'))
        try {
            this.$store.dispatch('dashboard/fetchQuizCount',this.userId)
            .then((response)=>{
                if(response.status==200){ 
                    this.quizCountdata={               
                        labels: response.data.map(x=>x.date.substr(0,10)),           
                        datasets: [
                            {
                                label: 'Participation(Date wise)',
                                backgroundColor: '#42424',
                                data: response.data.map(x=>x.count)                       
                            }
                        ]           
                    }  
                    this.loadQuiz=true               
                }
            })
            .catch((err)=>{
                console.log(err)
            })

            this.$store.dispatch('dashboard/fetchQuizUserCount',this.userId)
            .then((response)=>{
                if(response.status==200){ 
                    this.quizUserCountdata= {
                        labels: response.data.map(x=>x.quizTitle),
                        datasets: [
                            {
                                label: 'Participation(Topic wise)',
                                backgroundColor: '#42424',
                                data: response.data.map(x=>x.count)
                            }
                        ]
                    }     
                    this.loadUser=true 
                }
            })
            .catch((err)=>{
                console.log(err)
            })

            this.$store.dispatch('dashboard/fetchMonthCount',this.userId)
            .then((response)=>{
                if(response.status==200){ 
                    this.monthdata= {
                        labels: response.data.map(x=>x.month),
                        datasets: [
                            {
                                label: 'Login(Month wise)',
                                backgroundColor: '#42424',
                                data: response.data.map(x=>x.count)
                            }
                        ]
                    }     
                    this.loadMonth=true 
                }
            })
            .catch((err)=>{
                console.log(err)
            })

            this.$store.dispatch('dashboard/fetchBrowserCount',this.userId)
            .then((response)=>{
                if(response.status==200){ 
                    this.browserdata= {
                        labels: response.data.map(x=>x.browser),
                        datasets: [
                            {
                                label: 'Login(Browser wise)',
                                backgroundColor: '#42424',
                                data: response.data.map(x=>x.count)
                            }
                        ]
                    }     
                    this.loadBrowser=true 
                }
            })
            .catch((err)=>{
                console.log(err)
            })

            this.$store.dispatch('dashboard/fetchPlatformCount',this.userId)
            .then((response)=>{
                if(response.status==200){ 
                    this.platformdata= {
                        labels: response.data.map(x=>x.platform),
                        datasets: [
                            {
                                label: 'Login(Platform wise)',
                                backgroundColor: '#42424',
                                data: response.data.map(x=>x.count)
                            }
                        ]
                    }     
                    this.loadPlatform=true 
                }
            })
            .catch((err)=>{
                console.log(err)
            })

            this.$store.dispatch('dashboard/fetchDateCount',this.userId)
            .then((response)=>{
                if(response.status==200){ 
                    this.datedata= {
                        labels: response.data.map(x=>x.date.substr(0,10)),
                        datasets: [
                            {
                                label: 'Login(Date wise)',
                                backgroundColor: '#42424',
                                data: response.data.map(x=>x.count)
                            }
                        ]
                    }     
                    this.loadDate=true 
                }
            })
            .catch((err)=>{
                console.log(err)
            })

            this.chartOptions= {
                responsive: true,
                maintainAspectRatio: false
            }

            this.barChartOptions= {
                scales:{
                    yAxes: [{
						ticks: {
							beginAtZero: true
						},
						gridLines: {
							display: true
						}
					}],
                    xAxes: [{
						ticks: {
							beginAtZero: true
						},
						gridLines: {
							display: false
						}
					}]
                },
                responsive: true,
                maintainAspectRatio: false
            }
        } catch (error) {
            console.log(error)
        }
    },
    computed:{   
        roleName(){
            return this.userInfo.roleName
        },
        getCurrency(){
            return this.$store.getters['settings/currencySymbol']
        },
        stripePostUrl(){
            return config.hostname+'/pay-for-quiz'
        }    
    },
    created(){
        this.userInfo=this.$store.getters['dashboard/userInfo']
        this.$store.dispatch('dashboard/changeVisibility')
        this.$store.dispatch('dashboard/saveQuestionSerial',1)
        this.$store.dispatch('dashboard/saveQuizFlag',false)
        this.makePayment() 
        if(this.userInfo.roleName=='Student'){      
            this.initializeStudent()
        }else if(this.userInfo.roleName=='Admin'){
            this.initializeAdmin()
        }
    }
}
</script>