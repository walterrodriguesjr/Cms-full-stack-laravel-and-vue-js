<template>
    <div>
        <h1 ref="myref">Hook tutorial</h1>
        <h2>My name is: {{name}}</h2>
    </div>
</template>

<script>
export default {
    data(){
        return {
          name: 'sakek'  
        }
    },

    beforeCreate() {
        console.log(this.name);
    },

    created(){
        // chama principalmente os dados iniciais dele...
        console.log('data and other function is avalible but not html', this.name);

    },

    mounted() {
        console.log(this.$refs.myref)
    },
}
</script>