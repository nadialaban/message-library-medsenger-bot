<template>
    <div>
        <div>
            <h5>Библиотека сообщений</h5>

            <!-- фильтрация -->
            <div class="row" style="margin: 10px -5px;">
                <div class="col" v-if="is_admin">
                    <select class="form-control form-control-sm" v-model="current_clinic" @change="filter_messages()">
                        <option :value="undefined">Выбор клиники</option>
                        <option v-for="clinic in clinics" :value="clinic.id" v-if="clinic.name">{{ clinic.name }}</option>
                    </select>
                </div>

                <div class="col">
                    <select class="form-control form-control-sm" v-model="current_category" @change="filter_messages()">
                        <option :value="undefined">Выбор категории</option>
                        <option v-for="category in Object.keys(message_groups)" :value="category">{{ category }}</option>
                    </select>
                </div>
                <div class="col">
                    <input type="text" v-model="search_query" class="form-control form-control-sm"
                           placeholder="Поиск по сообщениям..." @change="filter_messages()">
                </div>
                <button class="btn btn-default btn-sm" style="margin-right: 5px" @click="create_message()">Создать</button>
            </div>

            <!-- сообщения -->
            <message-preview :message="message" :sent="sent_messages && sent_messages.includes(message.id)"
                             v-for="message in filtered_messages" v-if="filtered_messages.length" :key="`message_${message.id}`"/>

            <div v-else>
                <p style="text-align: center"><img :src="images.nothing_found"/></p>

                <p style="text-align: center">
                    <small>Ничего не найдено.</small>
                </p>
            </div>
        </div>
    </div>
</template>

<script>
import Card from "../common/Card.vue";
import MoreInfoBlock from "../common/MoreInfoBlock.vue";
import Loading from "../common/Loading.vue";

import MessagePreview from "./parts/MessagePreview.vue";

export default {
    name: "Dashboard",
    components: {MessagePreview, Loading, MoreInfoBlock, Card},
    props: {
        data: {
            required: true
        },
        clinics: {
            required: true
        }
    },
    data() {
        return {
            search_query: '',
            messages: [],
            sent_messages: [],
            filtered_messages: [],
            current_category: undefined,
            current_clinic: undefined
        }
    },
    computed: {
        message_groups() {
            return this.group_by(this.messages, 'category')
        }
    },
    methods: {
        filter_messages: function () {
            this.filtered_messages = []

            this.filtered_messages = this.messages.filter(m =>
                (!this.current_category || m.category == this.current_category) &&
                (m.title.toLowerCase().includes(this.search_query.toLowerCase()) ||
                    m.text.toLowerCase().includes(this.search_query.toLowerCase())))

            if (this.current_clinic)
                this.filtered_messages = this.filtered_messages.filter(m =>
                    !m.include_clinics && !m.exclude_clinics ||
                    m.include_clinics && m.include_clinics.includes(this.current_clinic) ||
                    m.exclude_clinics && !m.exclude_clinics.includes(this.current_clinic))

            this.filtered_messages = this.filtered_messages.sort((a, b) => ((a.title < b.title) ? -1 : ((a.title > b.title) ? 1 : 0)))
            this.$forceUpdate()
        },
        create_message: function () {
            Event.fire('navigate-to-create-message-page')
        },
    },
    created() {
        this.messages = this.data.messages
        this.filtered_messages = this.data.messages
        this.filtered_messages = this.filtered_messages.sort((a, b) => ((a.title < b.title) ? -1 : ((a.title > b.title) ? 1 : 0)))
        this.sent_messages = this.data.sent_messages
    },
    mounted() {
        this.messages = this.data.messages
        Event.listen('update-messages', (messages) => {
            this.messages = messages
            this.filter_messages()
            this.$forceUpdate()
        });
    }
}
</script>

<style scoped>
p {
    margin-top: 5px;
    margin-bottom: 5px;
}

small {
    font-size: 90%;
}
</style>
