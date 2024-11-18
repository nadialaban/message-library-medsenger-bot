<template>
    <div style="background-color: white;" >
        <div class="container">
            <div class="row">
                <div class="col-12">
                    <p><strong>Пациент:</strong> {{ patient.name }} ({{ patient.birthday }})</p>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: "DashboardHeader",
    props: ['patient'],
    methods: {
        go_home: function () {
            Event.fire('home');
        }
    }
}
</script>

<style scoped>
p {
    margin-top: 5px;
    margin-bottom: 5px;
    font-size: small;
}

h5 {
    margin-bottom: 10px;
    margin-top: 10px;
}
</style>
