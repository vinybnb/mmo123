<template>
    <div class="container-fluid">
        <account-form @reloadData="reloadData"></account-form>
        <account-edit-form @reloadData="reloadData" :account="currentAccount" v-if="currentAccount"></account-edit-form>
        <b-modal id="modal-confirm-delete" title="Xác nhận">
            <p class="my-4">Bạn chắc chắn muốn xóa chứ?</p>
            <div slot="modal-footer">
                <b-button squared variant="danger" @click="deleteAccount"><i class="fa fa-trash"></i> Xóa</b-button>
            </div>
        </b-modal>
        <b-modal id="modal-confirm-mass-delete" title="Xác nhận">
            <p class="my-4">Bạn chắc chắn muốn xóa chứ?</p>
            <div slot="modal-footer">
                <b-button squared variant="danger" @click="deleteAccounts"><i class="fa fa-trash"></i> Xóa</b-button>
            </div>
        </b-modal>
        <div class="row">
            <div class="col">
                <b-card>
                    <b-table
                        id="accounts-table"
                        ref="accountsTable"
                        hover
                        bordered
                        caption-top
                        responsive
                        selectable
                        :per-page="perPage"
                        :current-page="currentPage"

                        @row-selected="onRowSelected"
                        :items="accounts"
                        :fields="fields">
                        <template v-slot:table-caption>
                            <h2 class="text-center">Danh sách tài khoản</h2>
                            <div class="col">
                                <b-button @click="selectAllRows" squared variant="primary">All</b-button>
                                <b-button @click="clearSelected" squared variant="secondary">Clear</b-button>
                                <b-button variant="danger" squared @click="showMassDeleteConfirm()" :disabled="selectedAccounts.length == 0"><i class="fa fa-trash"> Xóa tài khoản</i></b-button>
                                <b-button v-b-modal.modal-account-form variant="success" squared class="float-right"><i class="fa fa-plus"> Thêm mới</i></b-button>
                            </div>
                        </template>
                        <template v-slot:cell(selected)="{ rowSelected }">
                            <template v-if="rowSelected">
                                <span>&check;</span>
                            </template>
                            <template v-else>
                                <span>&nbsp;</span>
                            </template>
                        </template>
                        <template v-slot:cell(index)="row">
                            {{ (currentPage - 1) * perPage + row.index + 1 }}
                        </template>
                        <template v-slot:cell(email)="row">
                            {{ row.item.email }} <template v-if="row.item.channel_name"><br/> <a :href="row.item.channel_link" target="_blank">{{ row.item.channel_name }}</a></template>
                        </template>
                        <template v-slot:cell(actions)="row">
                            <b-button-group>
                                <b-button squared variant="primary" @click="loginGmail(row.item)" title="Đăng nhập tự động"><i class="fa fa-sign-in"></i></b-button>
                                <b-button squared variant="secondary" @click="manualLogin(row.item)" title="Đăng nhập thủ công"><i class="fa fa-hand-o-right"></i></b-button>
                                <b-button squared variant="success" v-if="row.item.cookie" @click="loginByCookie(row.item)" title="Đăng nhập bằng cookie"><i class="fa fa-pie-chart"></i></b-button>
                                <b-button squared variant="warning" @click="showEditForm(row.item)" title="Sửa"><i class="fa fa-pencil"></i></b-button>
                                <b-button squared variant="danger" @click="showDeleteConfirm(row.item)" title="Xóa"><i class="fa fa-trash"></i></b-button>
                            </b-button-group>
                        </template>
                        <template v-slot:cell(status)="row">
                            <h4><b-badge variant="success" v-if="row.item.status == 1">OK</b-badge>
                            <b-badge variant="danger" v-else-if="row.item.status == 0">Lỗi</b-badge></h4>
                        </template>
                    </b-table>
                    <div class="float-right">
                        <b-pagination
                            v-model="currentPage"
                            :total-rows="rows"
                            :per-page="perPage"
                            aria-controls="accounts-table">
                        </b-pagination>
                    </div>
                </b-card>
            </div>
        </div>
    </div>
</template>

<script>
    import AccountForm from './AccountForm.vue'
    import AccountEditForm from './AccountEditForm.vue'

    export default {
        components: {
            AccountForm,
            AccountEditForm
        },
        data() {
            return {
                fields: [
                    {
                        key: 'selected',
                        label: 'Chọn'
                    },{
                        key: 'index',
                        label: '#'
                    },
                    {
                        key: 'email',
                        label: 'Email | Kênh'
                    },
                    {
                        key: 'notes',
                        label: 'Ghi chú'
                    },
                    {
                        key: 'actions',
                        label: 'Tác vụ'
                    },
                    {
                        key: 'status',
                        label: 'Trạng thái'
                    },
                    {
                        key: 'detail_reason',
                        label: 'Nguyên nhân chi tiết'
                    },
                    {
                        key: 'updated_at',
                        label: 'Cập nhật gần nhất'
                    },
                ],
                accounts: [],
                perPage: 10,
                currentPage: 1,
                currentAccount: null,
                selectedAccounts: []
            }
        },
        computed: {
            rows() {
                return this.accounts.length
            }
        },
        created() {
            this.loadAccounts()
        },
        methods: {
            async loadAccounts() {
                const response = await axios.get('/accounts')
                this.accounts = response.data.accounts
            },
            reloadData() {
                this.loadAccounts()
            },
            showEditForm(account) {
                this.currentAccount = account
                this.$bvModal.show('modal-account-edit-form')
            },
            showDeleteConfirm(account) {
                this.currentAccount = account
                this.$bvModal.show('modal-confirm-delete')
            },
            selectAllRows() {
                this.$refs.accountsTable.selectAllRows()
            },
            clearSelected() {
                this.$refs.accountsTable.clearSelected()
            },
            onRowSelected(items) {
                this.selectedAccounts = items
            },
            showMassDeleteConfirm() {
                this.$bvModal.show('modal-confirm-mass-delete')
            },
            async deleteAccounts() {
                const response = await axios.post('/accounts/mass-delete', {
                    ids: this.selectedAccounts.map(account => account.id)
                })

                if (response.data.status == 'success') {
                    this.$bvToast.toast(`Đã xóa thành công các tài khoản.`, {
                        title: 'Thành công',
                        autoHideDelay: 5000,
                        variant: 'success',
                        appendToast: true
                    })
                    this.$bvModal.hide('modal-confirm-mass-delete')
                    this.reloadData()
                } else {
                    this.$bvToast.toast(`Lỗi: ${response.data.message}.`, {
                        title: 'Lỗi',
                        autoHideDelay: 5000,
                        variant: 'danger',
                        appendToast: true
                    })
                }
            },
            async deleteAccount() {
                const response = await axios.post('/accounts/delete', {
                    id: this.currentAccount.id
                })

                if (response.data.status == 'success') {
                    this.$bvToast.toast(`Đã xóa thành công tài khoản.`, {
                        title: 'Thành công',
                        autoHideDelay: 5000,
                        variant: 'success',
                        appendToast: true
                    })
                    this.$bvModal.hide('modal-confirm-delete')
                    this.reloadData()
                } else {
                    this.$bvToast.toast(`Lỗi: ${response.data.message}.`, {
                        title: 'Lỗi',
                        autoHideDelay: 5000,
                        variant: 'danger',
                        appendToast: true
                    })
                }
            },
            async loginGmail(account) {
                const response = await axios.get('http://localhost:8080/login-gmail', {
                    params: {
                        email: account.email,
                        password: account.password,
                        recovery_email: account.recovery_email
                    }
                })
                if (response.data.Status == true) {
                    this.$bvToast.toast(`Đã đăng nhập công tài khoản.`, {
                        title: 'Thành công',
                        autoHideDelay: 5000,
                        variant: 'success',
                        appendToast: true
                    })
                    this.updateStatus(account.id, true, null, response.data.Cookie, response.data.Channel_Name, response.data.Channel_Link)
                } else {
                    this.$bvToast.toast(`Lỗi: ${response.data.Detail_Reason}.`, {
                        title: 'Đăng nhập lỗi',
                        autoHideDelay: 5000,
                        variant: 'danger',
                        appendToast: true
                    })
                    this.updateStatus(account.id, false, response.data.Detail_Reason)
                }
            },
            async manualLogin(account) {
                const response = await axios.get('http://localhost:8080/manual-login', {
                    params: {
                        email: account.email,
                        password: account.password
                    }
                })
                if (response.data.Status == true) {
                    this.$bvToast.toast(`Đã đăng nhập công tài khoản.`, {
                        title: 'Thành công',
                        autoHideDelay: 5000,
                        variant: 'success',
                        appendToast: true
                    })
                    this.updateStatus(account.id, true, null, response.data.Cookie, response.data.Channel_Name, response.data.Channel_Link)
                } else {
                    this.$bvToast.toast(`Lỗi: ${response.data.Detail_Reason}.`, {
                        title: 'Đăng nhập lỗi',
                        autoHideDelay: 5000,
                        variant: 'danger',
                        appendToast: true
                    })
                    this.updateStatus(account.id, false, response.data.Detail_Reason)
                }
            },
            async loginByCookie(account) {
                const response = await axios.get('http://localhost:8080/login-by-cookie', {
                    params: {
                        cookie: account.cookie
                    }
                })
                if (response.data.Status == true) {
                    this.$bvToast.toast(`Đã đăng nhập công tài khoản.`, {
                        title: 'Thành công',
                        autoHideDelay: 5000,
                        variant: 'success',
                        appendToast: true
                    })
                    this.updateStatus(account.id, true, null, response.data.Cookie)
                } else {
                    this.$bvToast.toast(`Lỗi: ${response.data.Detail_Reason}.`, {
                        title: 'Đăng nhập lỗi',
                        autoHideDelay: 5000,
                        variant: 'danger',
                        appendToast: true
                    })
                    this.updateStatus(account.id, false, response.data.Detail_Reason, response.data.Cookie)
                }
            },
            async updateStatus(id, status, detail_reason = null, cookie = null, channel_name = null, channel_link = null) {
                const response = await axios.post('/accounts/update-status', {
                    id: id,
                    status: status,
                    detail_reason: detail_reason,
                    cookie: cookie,
                    channel_name: channel_name,
                    channel_link: channel_link
                })

                if (response.data.status == 'success') {
                    this.$bvToast.toast(`Đã cập nhật trạng thái thành công.`, {
                        title: 'Thành công',
                        autoHideDelay: 5000,
                        variant: 'success',
                        appendToast: true
                    })
                } else {
                    this.$bvToast.toast(`Lỗi: ${response.data.message}.`, {
                        title: 'Lỗi',
                        autoHideDelay: 5000,
                        variant: 'danger',
                        appendToast: true
                    })
                }
                this.reloadData()
            }
        }
    }
</script>
