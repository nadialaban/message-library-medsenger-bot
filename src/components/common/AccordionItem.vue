<template>
    <div>
        <div class="accordion" v-if="title">
            <button class="btn btn-block accordion-button btn-right shadow-none collapsed" type="button" data-toggle="collapse" aria-expanded="false"
                    @click="collapsed = !collapsed" :data-target="`#collapse${object_id}`" :aria-controls="`collapse${object_id}`" :style="bg">
                <span v-if="is_admin" class="text-muted">{{ object_id }}&nbsp;–&nbsp;</span> {{title}} <span class="text-muted" v-if="sent">&nbsp;(отправлено)</span>
            </button>
            <div class="collapse" :id="`collapse${object_id}`" style="font-size: 14px">
                <card :image="image" additional_class="expanded">
                    <slot></slot>
                </card>
            </div>
        </div>
    </div>
</template>

<script>
import Card from "./Card.vue";

export default {
    name: "AccordionItem",
    props: ['title', 'image', 'object_id', 'sent'],
    components: {Card},
    data() {
        return {
            collapsed: true
        }
    },
    computed: {
        bg: function () {
            if (this.collapsed) {
                return "background-image: url('" + this.images.expand + "'); "
            } else {
                return "background-image: url('" + this.images.collapse + "'); "
            }
        }
    }
}
</script>

<style scoped>
.accordion-button {
    background-color: white;
    border-color: #006c88;
    text-align: left;
    padding: 10px 50px 10px 10px;
    color: #006c88;
    background-repeat: no-repeat;
    background-position: calc(100% - 10px) 50%;
    background-size: 25px;
}

.accordion-button:hover, .accordion-button:active, .accordion-button:focus, .accordion-button:not(.collapsed) {
    background-color: rgba(0, 108, 136, 0.1);
}

.accordion-button:not(.collapsed) {
    border-bottom-right-radius: 0;
    border-bottom-left-radius: 0;
}

.accordion-body a {
    font-size: 0.75rem;
}

</style>
