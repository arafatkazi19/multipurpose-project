<template>
    <div class="container">
        <div class="row mt-4" v-if="$gate.isAdminOrAuthor()">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <h3 class="card-title txt text-success">Users</h3>

                        <div class="card-tools">
                           <button class="btn btn-success" data-toggle="modal" @click="newModal">Add User
                               <i class="fas fa-user-plus fa-fw"></i>
                           </button>
                    </div>
                    </div>
                    <!-- /.card-header -->
                    <div class="card-body table-responsive p-0">
                        <table class="table table-hover text-nowrap">
                            <thead>
                            <tr>
                                <th>ID</th>
                                <th>Name</th>
                                <th>Email</th>
                                <th>Type</th>
                                <th>Registered At</th>
                                <th>Action</th>
                            </tr>
                            </thead>
                            <tbody>
                            <tr v-for="user in users.data" :key="user.id">
                                <td>{{user.id}}</td>
                                <td>{{user.name}}</td>
                                <td>{{user.email}}</td>
                                <td>{{user.type | upText}}</td>
                                <td>{{user.created_at | myDate}}</td>
                                <td>
                                    <a @click="editModal(user)">
                                    <i class="fa fa-edit yellow"></i>
                                    </a>

                                    <a @click="deleteUser(user.id)">
                                        <i class="fa fa-trash red"></i>
                                    </a>
                                </td>
                            </tr>

                            </tbody>
                        </table>
                    </div>
                    <!-- /.card-body -->
                </div>
                <!-- /.card -->
                <div class="card-footer">
                    <pagination class="justify-content-center" :data="users"
                                @pagination-change-page="getResults"></pagination>

                </div>
            </div>
        </div>

        <div  v-if="!$gate.isAdminOrAuthor()">
            <not-found></not-found>
        </div>
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNewLabel" aria-hidden="true">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title text-success" v-show="!editmode" id="exampleModalLabel">Add New</h5>
                        <h5 class="modal-title text-warning" v-show="editmode" id="exampleModalLabel">Update Info</h5>

                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>

                    <form @submit.prevent="editmode ? updateUser() : createUser()">
                    <div class="modal-body">
                        <div class="form-group">
                            <input v-model="form.name" type="text" name="name"
                                   placeholder="Name"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                            <has-error :form="form" field="name"></has-error>
                        </div>

                        <div class="form-group">
                            <input v-model="form.email" type="email" name="email"
                                   placeholder="Email"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                            <has-error :form="form" field="email"></has-error>
                        </div>

                        <div class="form-group">
                            <textarea v-model="form.bio" name="bio" id="bio"
                                   placeholder="Short Bio(Optional)"
                                      class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                            <has-error :form="form" field="bio"></has-error>
                        </div>

                        <div class="form-group">
                            <select v-model="form.type" type="text" id="type"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                                <option value="">SELECT USER ROLE</option>
                                <option value="admin">Admin</option>
                                <option value="user">Standard User</option>
                                <option value="author">Author</option>
                            </select>

                                <has-error :form="form" field="type"></has-error>
                        </div>

                        <div class="form-group">
                            <input v-model="form.password" type="password" name="password"
                                   placeholder="Password"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                            <has-error :form="form" field="password"></has-error>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                        <button v-show="!editmode" type="submit" class="btn btn-primary">Submit</button>
                        <button v-show="editmode" type="submit" class="btn btn-warning">Update</button>


                    </div>
                    </form>
                </div>
            </div>
        </div>
    </div>


</template>

<script>

    export default {
        name: "User",
        data() {

            return {
                editmode: false,
                users : {},
                // Create a new form instance
                form: new Form({
                    id:'',
                    name: '',
                    email: '',
                    password: '',
                    type:'',
                    bio:'',
                    photo:''

                })
            }
        },

        methods: {
            getResults(page = 1) {
                axios.get('api/user?page=' + page)
                    .then(response => {
                        this.users = response.data;
                    });
            },
            updateUser(){
                this.$Progress.start();
                //console.log("editing data");
                this.form.put('api/user/'+this.form.id)
                    .then(()=>{
                        $('#addNew').modal('hide');
                        swal.fire(
                            'Updated!',
                            'Your info has been updated',
                            'success'
                        )
                        this.$Progress.finish();
                        Fire.$emit('AfterCreate');
                    })
                    .catch(()=>{
                        this.$Progress.fail();
                    })
            },
            editModal(user){
                this.editmode = true;
                this.form.reset();
                $('#addNew').modal('show');
                this.form.fill(user);
            },
            newModal(){
                this.editmode = false;
                this.form.reset();
                $('#addNew').modal('show');
            },
            deleteUser(id){
                swal.fire({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    icon: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                }).then((result) => {
                    if (result.value) {
                        this.form.delete('api/user/' + id).then(() => {
                            swal.fire(
                                'Deleted!',
                                'Your file has been deleted.',
                                'success'
                            )
                            Fire.$emit('AfterCreate');
                        }).catch(() => {
                            swal.fire("failed", "There was something wrong", "warning");
                        });
                    }
                })
            },
            loadUsers(){
                if(this.$gate.isAdminOrAuthor()) {
                    axios.get("api/user").then(({data}) => (this.users = data));
                }
            },
            createUser(){
                this.$Progress.start();
                this.form.post('api/user')
                    .then(()=>{
                        Fire.$emit('AfterCreate');



                        // $('#addNew').on('hidden.bs.modal', function () {
                        //     $(this).find("input,textarea,select").val('').end();
                        //
                        //
                        // });

                        $('#addNew').modal('hide');
                        //location.reload();

                        // $('#addNew').on('hidden.bs.modal', function(){
                        //     $(this).find('form')[0].reset();
                        // });


                        Toast.fire({
                            icon: 'success',
                            title: 'User Created successfully'
                        });
                        this.$Progress.finish();
                    })
                    .catch(()=>{
                        this.$Progress.fail();
                        Toast.fire({
                            icon: 'error',
                            title: 'User not Created'
                        });
                    });


            }
        },
        created(){
            Fire.$on('searching',()=>{
                let query = this.$parent.search;
                axios.get('api/findUser?q=' + query)
                    .then((data)=>{
                        this.users = data.data
                    })
                    .catch(()=>{

                    })
            });
            this.loadUsers();
            //setInterval(()=>this.loadUsers(), 3000);
            Fire.$on('AfterCreate', ()=>{
               this.loadUsers();
            });
        }
    }

</script>

<style scoped>

</style>