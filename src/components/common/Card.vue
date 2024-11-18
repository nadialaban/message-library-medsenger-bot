<template>
    <div class="card" :class="additional_class" :style="bg">
        <div class="card-body">
            <h6 class="card-title" v-if="title">{{ title }}</h6>
            <slot></slot>
        </div>
        <slot name="footer"></slot>
    </div>
</template>

<script>
export default {
    name: "Card",
    props: ['title', 'image', 'additional_class'],
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
.card {
    margin-bottom: 5px;
    background-repeat: no-repeat;
    background-position: right bottom;
    background-size: 55px;
    border-color: rgba(0, 108, 136, 0.5);
}

.card a {
    font-size: 0.75rem;
}

.card-title {
    margin-bottom: 1.5rem;
}

.card-body {
    padding: 10px;
}

.expanded {
    border-bottom-left-radius: 0;
    border-top-left-radius: 0;
    border-top-right-radius: 0;
    border-top: 0;
    border-color:  #006c88;
    margin: 0;
}
</style>
