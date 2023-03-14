
import ContactList from './ContactList.vue';

<template>
    <div class="page row">
        <div class="col-md-10">
            <InputSearch v-model="searchText" />
        </div>
        <div class="mt-3 col-md-6">
            <h4>
                Danh bạ
                <i class="fas fa-address-book"></i>
            </h4>
            <ContactList
                v-if="filteredContactsCount > 0"
                :contact="filteredContacts"
                v-model:activeIndex="activeIndex"
            />
            <p v-else>Không có liên hệ nào</p>

            <div class="mt3 row justify-content-around align-items-center">
                <button class="btn btn-sm btn-primary" @click="refreshList()">
                    <i class="fas fa-redo"></i> Làm mới
                </button>

                <button class="btn btn-sm btn-succes" @click="goToAddContact">
                    <i class="fas fa-plus"></i> Thêm mới
                </button>

                <button class="btn btn-sm bth-danger" @click="removeAllContacts">
                    <i class="fas fa-trash"></i> Xóa tất cả
                </button>
            </div>
        </div>
        <div class="mt-3 col-md-6">
            <div v-if="activceContact">
                <h4>
                    Chi tiết liên hệ
                    <i class="fas fa-address-card"></i>
                </h4>
                <ContactCard :contact="activceContact" />
            </div>
        </div>
    </div>
</template>

<script>
import ContactCard from "@/components/ContactCard.vue";
import InputSearch from "@/comnponents/InputSearch.vue";
import ContactList from "@/comnponents/ContactList.vue";
import ContactService from "@/services/contact.service";

export default {
    components: {
        ContactCard,
        InputSearch,
        ContactList,
    },
    data() {
        return {
            contacts: [],
            activeIndex: -1,
            searchText: "",
        };
    },
    watch: {
        //Giám sát các thay đổi của biến searchtext
        //Bỏ chọn ptử đang được chọn trong danh sách
        searchText() {
            this.activeIndex = -1;
        },
    },
    computed: {
        //Chuyển đổi các đổi tượng contact thành chuối để tiện cho tìm kiếm
        contactString(){
            return this.contacts.map((contact) => {
                const { name, email, address, phone } = contact;
                return [name, email, address, phone].join("");
            });

        },
        filteredContacts() {
            if (!this.searchText) return this.contacts;
            return this.contacts.filter((_contact, index) =>
                this.contactString[index].includes(this.searchText)
            );
        },
        activceContact() {
            if (this.activeIndex <0) return null;
            return this.filteredContacts[this.activeIndex];
        },
        filteredContactsCount() {
            return this.filteredContacts.length;
        },
    },
    methods: {
        async retrieveContacts() {
            try {
                this.contacts = await ContactService.getAll();
            } catch (error) {
                console.log(error);
            }
        },

        refreshList() {
            this.retrieveContacts();
            this.activeIndex = -1;
        },

        async removeAllContacts() {
            if (confirm("Bạn muốn xóa hết tất cả liên hệ?")){
                try {
                    await ContactService.deleteAll();
                    this.refreshList();
                } catch (error) {
                console.log(error);
                }
            }
        },

        goToAddContact() {
            this.$router.push({ name: "contact.add"});
        },
    },
    mounted() {
        this.refreshList();
    },
};
</script>

<style scoped>
.page {
    text-align: left;
    max-width: 750px;
}
</style>