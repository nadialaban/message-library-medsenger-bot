<template>
    <div class="accordion-item">
        <h2 class="accordion-header" :id="`heading${object_id}`">
            <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" :data-bs-target="`#collapse${object_id}`"
                    aria-expanded="false" :aria-controls="`collapse${object_id}`">
                {{ title }}
            </button>
        </h2>
        <div :id="`collapse${object_id}`" class="accordion-collapse collapse" :aria-labelledby="`heading${object_id}`"
             :data-bs-parent="parent">
            <div class="accordion-body" :style="bg">
                <slot></slot>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: "AccordionItem",
    props: ['title', 'image', 'object_id', 'parent'],
    computed: {
        bg: function () {
            if (this.image) {
                return "background-image: url('" + this.image + "'); "
            } else {
                return "background-image: none;"
            }
        }
    }
}
</script>

<style scoped>
.accordion-body {
    margin-bottom: 5px;
    background-repeat: no-repeat;
    background-position: right bottom;
    background-size: 55px;
}

.accordion-body a {
    font-size: 0.75rem;
}

</style>
