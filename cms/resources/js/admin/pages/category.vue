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
            Category <Button @click="addModal=true"><Icon type="md-add" />Add category</Button>
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
              <tr v-for="(tag, i) in tags" :key="i" v-if="tags.length">
                <td>{{tag.id}}</td>
                <td class="_table_name">{{tag.tagName}}</td>
                <td>{{tag.created_at}}</td>
                <td>
                  <Button type="info" size="small" @click="showEditModal(tag, i)">Edit</Button>
                  <Button type="error" size="small" @click="showDeletingModal(tag, i)" :loading="tag.isDeleting">Delete</Button>
                </td>
              </tr>
              <!-- ITEMS -->
            </table>
          </div>
        </div>

        <!-- tag adding modal -->
        <Modal
          v-model="addModal"
          title="Add category"
          :mask-closable="false"
          :closable="false"
        >

            <Input v-model="data.categoryName" placeholder="Add category name" />
            <div class="space"></div>
             <Upload
                        red="uploads"
                        type="drag"
                        :headers="{'x-csrf-token' : token, 'X-Requested-With' : 'XMLHttpRequest'}"
                        :on-success="handleSuccess"
                        :on-error="handleError"
                        :format="['jpg','jpeg','png']"
                        :max-size="2048"
                        :on-format-error="handleFormatError"
                        :on-exceeded-size="handleMaxSize"
                        action="/app/upload">
                        <div style="padding: 20px 0">
                            <Icon type="ios-cloud-upload" size="52" style="color: #3399ff"></Icon>
                            <p>Click or drag files here to upload</p>
                        </div>
                    </Upload>
                    <div class="demo-cloud-upload" v-if="data.iconImage">

                      <img :src="`/uploads/${data.iconImage}`" />
                      <div class="demo-upload-list-cover">
                        <Icon type='ios-trash-outline' @click="deleteImage"></Icon>
                      </div>
                    </div>

          <div slot="footer">
            <Button type="default" @click="addModal=false">Close</Button>
            <Button type="primary" @click="addCategory" :disable="isAdding" :loading="isAdding">{{isAdding ? 'Adding..' : 'Add Category'}}</Button>
          </div>
        </Modal>

        <!-- tag editing modal -->
        <Modal
          v-model="editModal"
          title="Edit tag"
          :mask-closable="false"
          :closable="false"
        >
            <Input v-model="editData.tagName" placeholder="Edit tag name" />

          <div slot="footer">
            <Button type="default" @click="editModal=false">Close</Button>
            <Button type="primary" @click="editTag" :disable="isAdding" :loading="isAdding">{{isAdding ? 'Editing..' : 'Edit tag'}}</Button>
          </div>
        </Modal>

        <!-- deleting alert modal -->
         <Modal v-model="showDeleteModal" width="360">
            <template #header>
                <p style="color:#f60;text-align:center">
                    <Icon type="ios-information-circle"></Icon>
                    <span>Delete confirmation</span>
                </p>
            </template>
            <div style="text-align:center">
                <p>Are you sure you want to delete tag?</p>
                
            </div>
            <template #footer>
                <Button type="error" size="large" long :loading="isDeleing" :disabled="isDeleing" @click="deleteTag">Delete</Button>
            </template>
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
        iconImage: '',
        categoryName: ''
      },
      addModal: false,
      editModal: false,
      isAdding: false,
      tags:[],
      editData: {
        tagName: ''
      },
      index: -1,
      showDeleteModal: false,
      isDeleing: false,
      deleteItem: {},
      deletingIndex: -1,
      token: ''

    }
  },

  //métodos de adicionar tag
  methods: {
    async addCategory(){
      if(this.data.categoryName.trim()=='') return this.e('Category name is required')
      if(this.data.iconImage.trim()=='') return this.e('Icon image is required')
      
      const res = await this.callApi('post', 'app/create_category', this.data)
      if(res.status===201){
        this.tags.unshift(res.data)
        this.s('Category has been added successully!')
        this.addModal = false
        this.data.categoryName = ''
        this.data.iconImage = ''
      }else{
        //validação do Add tag, vinculada a validação de AdminController
        if(res.status==422){
          if(res.data.errors.categoryName){
            this.e(res.data.errors.categoryName[0])
          }
          if(res.data.errors.iconImage){
            this.e(res.data.errors.iconImage[0])
          }
          console.log(res.data.errors.tagName);
        }else{
          this.swr()
        }
      }
    },

    //métodos de editar tag
    async editTag(){
      if(this.editData.tagName.trim()=='') return this.e('Tag name is required')
      const res = await this.callApi('post', 'app/edit_tag', this.editData)
      if(res.status===200){
          this.tags[this.index].tagName = this.editData.tagName
        this.s('Tag has been edited successully!')
        this.editModal = false
      }else{
        //validação do Add tag, vinculada a validação de AdminController
        if(res.status==422){
          if(res.data.errors.tagName){
            this.e(res.data.errors.tagName[0])
          }
        }else{
          this.swr()
        }
      }
    },

    showEditModal(tag, index){
      let obj = {
        id: tag.id,
        tagName: tag.tagName
      }
      this.editData = obj
      this.editModal = true
      this.index = index
    },

    async deleteTag(){
      this.isDeleing = true
      const res = await this.callApi('post', 'app/delete_tag', this.deleteItem)
      if(res.status===200){
        this.tags.splice(this.deletingIndex, 1)
        this.s('Tag has been deleted successfully!')
      }else{
        this.swr()
      }
        this.isDeleing = false
        this.showDeleteModal = false
    },

    showDeletingModal(tag, i){
      this.deleteItem = tag
      this.deletingIndex = i
      this.showDeleteModal = true
    },

    handleSuccess (res, file) {
            this.data.iconImage = res     
            },
    handleError (res, file) {
            this.$Notice.warning({
                    title: 'The file format is incorrect',
                    desc: `${file.errors.file.length ? file.errors.file.length[0] : 'Something went wrong!'}`
                });
            },
            handleFormatError (file) { 
                this.$Notice.warning({
                    title: 'The file format is incorrect',
                    desc: 'File format of ' + file.name + ' is incorrect, please select jpg or png.'
                });
            },
            handleMaxSize (file) {
                this.$Notice.warning({
                    title: 'Exceeding file size limit',
                    desc: 'File  ' + file.name + ' is too large, no more than 2M.'
                });
            },
            async deleteImage(){
              let image = this.data.iconImage;
              this.data.iconImage = ''
              this.$refs.uploads.clearFiles()
              const res = await this.callApi('post', 'app/delete_image', {imageName: image})
              if(res.status!=200){
                this.data.iconImage = image
                this.swr()
              }
            }
  },
  

  async created(){
    this.token = window.Laravel.csrfToken
    const res = await this.callApi('get', 'app/get_tags')
    if(res.status==200){
      this.tags = res.data
    }else{
      this.swr()
    }
  }
}
</script>