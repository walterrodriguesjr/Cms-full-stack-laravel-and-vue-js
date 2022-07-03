<template>
  <div>
    <div class="content">
      <div class="container-fluid">
        <!--~~~~~~~ TABLE ONE ~~~~~~~~~-->
        <div
          class="
            _1adminOverveiw_table_recent
            _box_shadow
            _border_radious
            _mar_b30
            _p20
          "
        >
          <p class="_title0">
            Tags <Button @click="addModal=true"><Icon type="md-add" />Add tag</Button>
          </p>

          <div class="_overflow _table_div">
            <table class="_table">
              <!-- TABLE TITLE -->
              <tr>
                <th>ID</th>
                <th>Tag name</th>
                <th>Created_at</th>
                <th>Action</th>
              </tr>
              <!-- TABLE TITLE -->

              <!-- ITEMS -->
              <tr>
                <td>1</td>
                <td class="_table_name">
                  Manhattan's art center "Shed" opening ceremony
                </td>
                <td>Economy</td>
                <td>
                  <Button type="info" size="small">Info</Button>
                  <Button type="error" size="small">Error</Button>
                </td>
              </tr>
              <!-- ITEMS -->
            </table>
          </div>
        </div>

        <!-- tag adding modal -->
        <Modal
          v-model="addModal"
          title="Add tag"
          :mask-closable="false"
          :closable="false"
        >
            <Input v-model="data.tagName" placeholder="Ad tag name" />

          <div slot="footer">
            <Button type="default" @click="addModal=false">Close</Button>
            <Button type="primary" @click="addTag" :disable="isAdding" :loading="isAdding">{{isAdding ? 'Adding..' : 'Add tag'}}</Button>
          </div>
        </Modal>

      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      data: {
        tagName: ''
      },
      addModal: false,
      isAdding: false,
    }
  },

  methods: {
    async addTag(){
      if(this.data.tagName.trim()=='') return this.e('Tag name is required')
      const res = await this.callApi('post', 'app/create_tag', this.data)
      if(res.status===201){
        this.s('Tag has been added successully!')
        this.addModal = false
      }else{
        this.swr()
      }
    }
  }
}
</script>