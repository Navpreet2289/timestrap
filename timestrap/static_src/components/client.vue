<template>
<div class="client">
    <div class="row py-2 bg-secondary text-white">
        <template v-if="edit">
            <div class="col-10">
                <input name="client-name"
                       type="text"
                       class="form-control form-control-sm"
                       v-model.trim="name"
                       v-on:keyup.enter="saveClient"
                       required />
            </div>
            <div class="col-2">
                <button name="client-save"
                        class="btn btn-success btn-sm w-100"
                        v-block-during-fetch
                        v-on:click="saveClient">
                    Save
                </button>
            </div>
        </template>

        <template v-else>
            <div v-bind:class="['col-sm-4', 'd-flex', 'align-items-center']">
                <i class="fa fa-address-book mr-2" aria-hidden="true"></i>
                <span class="font-weight-bold text-uppercase client-name">{{ client.name }}</span>
            </div>
            <div class="col-sm-2 d-flex align-items-center">
                <i class="fa fa-clock-o mr-2" aria-hidden="true"></i>
                <strong>Total Time</strong>
            </div>
            <div class="col-sm-2 d-flex align-items-center">
                <i class="fa fa-list mr-2" aria-hidden="true"></i>
                <strong>Entries</strong>
            </div>
            <div class="col-sm-3 d-flex align-items-center">
                <i class="fa fa-percent mr-2" aria-hidden="true"></i>
                <strong>Progress</strong>
            </div>
            <div class="col-sm-1 d-flex align-self-center justify-content-end">
                <template v-if="this.$perms.change_client || this.$perms.delete_client">
                    <button name="client-menu"
                            class="btn btn-faded btn-sm btn-icon dropdown-toggle"
                            type="button"
                            data-toggle="dropdown"
                            aria-haspopup="true"
                            aria-expanded="false">
                        <i class="fa fa-ellipsis-v" aria-hidden="true"></i>
                    </button>
                    <div class="dropdown-menu dropdown-menu-right"
                         aria-labelledby="entry-menu">
                        <a id="client-menu-change"
                           class="dropdown-item"
                           href="#"
                           v-if="this.$perms.change_client"
                           v-on:click.prevent
                           v-on:click="toggleEditModal(client, index)">Edit</a>
                        <a id="client-menu-delete"
                           class="dropdown-item"
                           href="#"
                           v-if="this.$perms.delete_client"
                           v-on:click.prevent
                           v-on:click="deleteClient">Delete</a>
                        <a id="client-menu-archive"
                           class="dropdown-item"
                           href="#"
                           v-if="this.$perms.change_client"
                           v-on:click.prevent
                           v-on:click="archiveClient">Archive</a>
                    </div>
                </template>
            </div>
        </template>
    </div>

    <template v-if="this.$perms.view_project">
        <project v-for="(project, project_index) in client.projects"
                 @removeProject="removeProject"
                 v-bind:project="project"
                 v-bind:index="project_index"
                 v-bind:client_index="index"
                 v-bind:key="project.id"
                 v-bind:toggleEditModal="toggleProjectEditModal"></project>
    </template>

</div>
</template>

<script>
const Project = require('./project.vue');

export default {
    props: ['client', 'index', 'key', 'toggleEditModal', 'toggleProjectEditModal', 'removeProject'],
    data() {
        return {
            edit: false,
            name: this.client.name
        };
    },
    methods: {
        deleteClient() {
            this.$quickFetch(this.client.url, 'delete').then(function(response) {
                if (response.status === 204) {
                    $.growl.notice({ message: 'Client deleted!' });
                    this.$emit('removeClient', this.index);
                } else {
                    $.growl.error({ message: 'Client delete failed ):' });
                }
            }.bind(this));
        },
        archiveClient() {
            const body = {
                name: this.client.name,
                archive: true
            };
            this.$quickFetch(this.client.url, 'put', body).then(data => {
                if (data.id) {
                    $.growl.notice({ message: 'Client archived!' });
                    this.$emit('removeClient', this.index);
                }
            }).catch(error => console.log(error));
        }
    },
    components: {
        Project
    }
};
</script>
