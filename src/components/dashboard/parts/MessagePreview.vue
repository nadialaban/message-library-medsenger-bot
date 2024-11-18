<template>
    <div class="row">
        <div style="width: 100px" v-if="!mobile">
            <button class="btn shadow-none btn-primary btn-sm w-100 btn-left" style="height: 100%" @click="send_now(message)">
                Отправить
            </button>
        </div>
        <div style="width: 50px" v-else>
            <button class="btn shadow-none btn-primary btn-sm w-100 btn-left" style="height: 100%" @click="send_now(message)">
                <img :src="images.send" height="25"/>
            </button>
        </div>

        <!-- сообщение -->
        <div class="col">
            <div class="accordion" :id="`message_${message.id}`">
                <accordion-item :title="message.title" :image="images.message" :sent="sent"
                                :parent="`#message_${message.id}`" :key="message.id" :object_id="message.id">
                    <span class="card-text" v-html="message.text.replaceAll('\n','<br>')"/>

                    <!-- Файлы -->
                    <div class="row">
                        <div v-for="file in message.attached_files" class="col-5">

                                <a href="#" @click="get_file(file.id, 'download')"><img :src="images.file" height="20"/>{{ file.name }} (скачать)</a>
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

                    <div v-if="message.pin_files">
                        <small><i>
                            Закрепляет файлы
                            <ul>
                                <li v-for="file in message.attached_files" v-if="file.title">
                                    {{ file.title }} ({{ file.name }})
                                </li>
                            </ul>

                        </i></small>
                    </div>

                    <!-- Действия -->
                    <div style="margin-top: 15px">
                        <a href="#" @click="edit_message(message)"
                           v-if="is_admin || message.editors.includes(doctor_id)">Редактировать</a>
                        <a href="#" @click="delete_message(message)" v-if="is_admin">Удалить</a>
                    </div>

                </accordion-item>
            </div>
        </div>
    </div>
</template>

<script>
import Loading from "../../common/Loading.vue";
import MoreInfoBlock from "../../common/MoreInfoBlock.vue";
import AccordionItem from "../../common/AccordionItem.vue";
import downloadjs from "downloadjs";

export default {
    name: "MessagePreview",
    components: {AccordionItem, MoreInfoBlock, Loading},
    props: ['message', 'sent'],
    data() {
        return {
            files_to_show: {}
        }
    },

    methods: {
        send_now: function (message) {
            let alert = () => {
                this.sent_messages.push(message.id)
                this.$alert("Сообщение отправлено!");
            }

            this.$confirm({
                message: `Отправить сообщение ` + message.title + ` пациенту прямо сейчас?`,
                button: {
                    no: 'Нет',
                    yes: 'Да'
                },
                callback: confirm => {
                    if (confirm)
                        this.axios
                            .get(this.url('/api/send_message/' + message.id))
                            .then(alert);
                }
            })
        },
        edit_message: function (message) {
            Event.fire('edit-message', message)
        },
        delete_message: function (message) {
            this.$confirm({
                message: `Вы уверены, что хотите удалить сообщение ` + message.title + `?`,
                button: {
                    no: 'Нет',
                    yes: 'Да, удалить'
                },
                callback: confirm => {
                    if (confirm)
                        this.axios
                            .post(this.url('/api/delete_message'), message)
                            .then(this.process_delete_message_answer);

                }
            })
        },
        process_delete_message_answer: function (response) {
            if (response.data.deleted_id)
                Event.fire('remove-message', response.data.deleted_id)
        },
        get_file: function (file_id, action) {
            this.axios
                .get(this.url('/api/get_file/' + file_id))
                .then(response => {
                    let file = response.data
                    if (action == 'download')
                        downloadjs(`data:${file.type};base64,${response.data.base64}`, file.name, file.type);
                    else if (action == 'show')
                        this.files_to_show[file.id] = file
                    this.$forceUpdate()
                })
                .catch(() => Event.fire('load-error'));
        },
    },
    computed: {
        img_width() {
            return Math.floor(window.innerWidth * 0.4)
        },
        img_height() {
            return Math.floor(window.innerHeight * 0.5)
        },
    },
    mounted() {

        Event.listen('open-more-info', (id) => {
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
.col {
    padding: 0;
}

p {
    margin-top: 5px;
    margin-bottom: 5px;
}

img {
    object-fit: contain;
    object-position: left top;
}
</style>
