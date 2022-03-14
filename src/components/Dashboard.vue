<template>
    <div>
        <div>
            <h3>Библиотека сообщений</h3>

            <!-- фильтрация -->
            <div class="row" style="margin-bottom: 5px;">
                <div class="col" v-if="is_admin">
                    <select class="form-control form-control-sm" v-model="current_clinic">
                        <option :value="undefined" disabled>Выбор клиники</option>
                        <option v-for="clinic in clinics" :value="clinic.id">{{
                                clinic.name
                            }}
                        </option>
                    </select>
                </div>

                <div class="col">
                    <select class="form-control form-control-sm" v-model="current_category">
                        <option :value="undefined" disabled>Выбор категории</option>
                        <option v-for="category in Object.keys(message_groups)" :value="category">{{
                                category
                            }}
                        </option>
                    </select>
                </div>
                <div class="col">
                    <input type="text" v-model="search_query" class="form-control form-control-sm"
                           placeholder="Поиск по сообщениям...">
                </div>
                <div class="d-grid col-1 mx-auto">
                    <button class="btn btn-primary btn-sm" @click="create_message()">Создать</button>
                </div>
            </div>

            <br>

            <!-- карточки -->
            <div class="accordion" id="accordionMessages" v-if="filtered_messages.length">
                <accordion-item :title="message.title" :image="images.message" parent="#accordionMessages" :key="i"
                                :object_id="i"
                                v-for="(message, i) in filtered_messages">

                    <p class="card-text"> {{ message.text }} </p>

                    <!-- Файлы -->
                    <div class="row">
                        <div v-for="file in message.attached_files" class="col-6">
                            <div class="row-fluid">
                                <img :src="images.file" height="20"/>
                                <a href="#" @click="get_file(file.id, 'download')">{{ file.name }} (скачать)</a>
                            </div>
                        </div>
                    </div>

                    <div v-for="file in message.attached_files">
                        <more-info-block :title="`Просмотр ${file.name}`" :id="`file_${file.id}`"
                                         v-if="file.type.includes('image')">
                            <loading v-if="!files_to_show[file.id]"/>
                            <img :src="`data:${file.type};base64,${files_to_show[file.id].base64}`"
                                 :style="`max-width: ${img_width}px; max-height: ${img_height}px;`" v-else/>
                        </more-info-block>
                    </div>

                    <!-- Действия -->
                    <div style="margin-top: 15px">
                        <button class="btn btn-primary btn-sm" @click="send_now(message)">Отправить</button>
                        <a href="#" @click="edit_message(message)"
                           v-if="is_admin || message.editors.includes(doctor_id)">Редактировать</a>
                        <a href="#" @click="delete_message(message)" v-if="is_admin">Удалить</a>
                    </div>

                </accordion-item>
            </div>

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
import Card from "./parts/Card";
import MoreInfoBlock from "./parts/MoreInfoBlock";
import Loading from "./parts/Loading";
import AccordionItem from "./parts/AccordionItem";

import downloadjs from "downloadjs";

export default {
    name: "Dashboard",
    components: {AccordionItem, Loading, MoreInfoBlock, Card},
    props: {
        messages: {
            required: true
        },
        clinics: {
            required: true
        }
    },
    data() {
        return {
            search_query: '',
            files_to_show: {},
            current_category: undefined,
            current_clinic: undefined
        }
    },
    computed: {
        filtered_messages() {
            let messages = []
            if (!this.current_category)
                messages = this.messages.filter(m =>
                    m.title.toLowerCase().includes(this.search_query.toLowerCase()) ||
                    m.text.toLowerCase().includes(this.search_query.toLowerCase()))
            else
                messages = this.message_groups[this.current_category].filter(m =>
                    m.title.toLowerCase().includes(this.search_query.toLowerCase()) ||
                    m.text.toLowerCase().includes(this.search_query.toLowerCase()))

            if (this.current_clinic)
                messages = messages.filter(m => !m.include_clinics && !m.exclude_clinics ||
                    m.include_clinics && m.include_clinics.includes(this.current_clinic) ||
                    m.exclude_clinics && m.exclude_clinics.includes(this.current_clinic))

            return messages
        },
        message_groups() {
            return this.group_by(this.messages, 'category')
        },
        img_width() {
            return Math.floor(window.innerWidth * 0.4)
        },
        img_height() {
            return Math.floor(window.innerHeight * 0.5)
        },
    },
    methods: {
        get_file: function (file_id, action) {
            this.axios.get(this.url('/api/get_file/' + file_id)).then(response => {
                let file = response.data
                if (action == 'download')
                    downloadjs(`data:${file.type};base64,${response.data.base64}`, file.name, file.type);
                else if (action == 'show')
                    this.files_to_show[file.id] = file
                this.$forceUpdate()
            }).catch(() => MyEvent.fire('load-error'));
        },
        // messages
        create_message: function () {
            MyEvent.fire('navigate-to-create-message-page')
        },
        edit_message: function (message) {
            MyEvent.fire('edit-message', message)
        },
        delete_message: function (message) {
            this.$confirm(
                {
                    message: `Вы уверены, что хотите удалить сообщение ` + message.title + `?`,
                    button: {
                        no: 'Нет',
                        yes: 'Да, удалить'
                    },
                    callback: confirm => {
                        if (confirm) {
                            this.axios.post(this.url('/api/delete_message'), message)
                                .then(this.process_delete_message_answer);
                        }
                    }
                }
            )
        },
        process_delete_message_answer: function (response) {
            if (response.data.deleted_id) {
                MyEvent.fire('remove-message', response.data.deleted_id)
            }
        },
        send_now: function (message) {
            let alert = () => {
                this.$alert("Сообщение отправлено!");
            }

            this.$confirm(
                {
                    message: `Отправить сообщение ` + message.title + ` пациенту прямо сейчас?`,
                    button: {
                        no: 'Нет',
                        yes: 'Да'
                    },
                    callback: confirm => {
                        if (confirm) {
                            this.axios.get(this.url('/api/send_message/' + message.id)).then(alert);
                        }
                    }
                }
            )
        },

    },
    mounted() {
        MyEvent.listen('dashboard-to-main', () => {
            if (this.window_mode == 'settings') {
                this.state = 'main'
            }
        });

        MyEvent.listen('home', () => {
            this.state = 'main'
        });

        MyEvent.listen('open-more-info', (id) => {
            if (id.includes('file')) {
                let file_id = parseInt(id.split('_')[1])
                if (!this.files_to_show[id]) {
                    this.get_file(file_id, 'show')
                }
            }
        })

    }
}
</script>

<style scoped>
p {
    margin-top: 5px;
    margin-bottom: 5px;
}

h5 {
    margin-bottom: 10px;
    margin-top: 10px;
    font-size: 1.15rem;
}

.sidebar {
    margin: -5px 10px -5px -5%;
    background-color: #24a8b4;
    color: #fcfcfc;
}

small {
    font-size: 90%;
}

img {
    object-fit: contain;
    object-position: left top;
}
</style>
