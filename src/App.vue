<template>
    <div style="padding-bottom: 15px;">
        <vue-confirm-dialog/>
        <loading v-if="state == 'loading'"/>
        <div v-else>
            <dashboard-header :patient="patient" v-if="patient"/>
            <div class="container">
                <dashboard :clinics="clinics" :data="{'messages': messages, 'sent_messages': patient.sent_messages}"
                           v-show="state == 'dashboard'"/>
                <message-editor :clinics="clinics" v-show="state == 'message-editor'"/>
            </div>
        </div>
    </div>
</template>

<script>
import Dashboard from "./components/dashboard/Dashboard.vue";
import Loading from "./components/common/Loading";
import MessageEditor from "./components/MessageEditor";
import DashboardHeader from "./components/dashboard/parts/DashboardHeader.vue";

export default {
    name: 'app',
    components: {DashboardHeader, MessageEditor, Loading, Dashboard},
    data() {
        return {
            state: "loading",
            patient: undefined,
            message: {},
            messages: [],
            clinics: []
        }
    },
    created() {
        console.log("running created");

        // navigation
        Event.listen('navigate-to-create-message-page', () => this.state = 'message-editor');
        Event.listen('back-to-dashboard', () => this.state = 'dashboard');

        // message managing
        Event.listen('message-created', (data) => {
            this.state = 'dashboard'
            this.messages.push(data)
            Event.fire('update-messages', this.messages);
        });
        Event.listen('message-edited', (data) => {
            this.messages.forEach((m) => {
                if (m.id == data.id) {
                    console.log(m)
                    // m = data
                    this.$forceUpdate()
                }
            })
            let index = this.messages.findIndex(m => m.id == data.id)
            if (index != null) {
                console.log(1)
                this.messages[index] = data
                this.$forceUpdate()
            }
            this.state = 'dashboard'
            Event.fire('update-messages', this.messages);
        });
        Event.listen('edit-message', (message) => {
            this.state = 'message-editor'
            this.axios.get(this.url('/api/get_message/' + message.id)).then(response => {
                Event.fire('navigate-to-edit-message-page', response.data);
            })
        });

        Event.listen('remove-message', (id) => {
            this.messages = this.messages.filter(m => m.id != id)
            Event.fire('update-messages', this.messages);
        })

        this.load()
    },
    mounted() {
        this.load()
    },
    methods: {
        load: function () {
            if (this.window_mode == 'done') {
                this.state = 'done'
            }

            if (this.window_mode == 'settings') {
                this.axios.get(this.url('/api/get_patient')).then((response) => {
                    this.patient = response.data
                    this.axios.get(this.url('/api/get_available_messages')).then(this.process_load_answer);
                })
            }
        },
        process_load_answer: function (response) {
            if (this.window_mode == 'settings') {
                this.messages = response.data;
                this.state = 'dashboard';
            }

            if (this.is_admin) {
                this.axios
                    .get(this.url('/api/get_clinics_info'))
                    .then(response =>
                        this.clinics = response.data
                            .filter((c) => c.name)
                            .sort((a, b) => ((a.name < b.name) ? -1 : ((a.name > b.name) ? 1 : 0)))
                    )
            }
        },
        process_load_error: function (response) {
            this.state = 'load-error'
        }
    }
}
</script>

<style>
.card {
    border-color: rgba(0, 108, 136, 0.5);
}

.row {
    margin: 5px 0;
}


.btn-right {
    border-top-left-radius: 0;
    border-bottom-left-radius: 0;
}

.btn-left {
    border-top-right-radius: 0;
    border-bottom-right-radius: 0;
}

input[type=checkbox] {
    /* Double-sized Checkboxes */
    -ms-transform: scale(1.2); /* IE */
    -moz-transform: scale(1.2); /* FF */
    -webkit-transform: scale(1.2); /* Safari and Chrome */
    -o-transform: scale(1.2); /* Opera */
    transform: scale(1.2);
    margin: 10px;
}
</style>

