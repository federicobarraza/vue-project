<template>
  <div class="contact-page">
    <h3>Consulta de Contactos</h3>
    <contact-icon></contact-icon>
    <div class="menu">
      <a href="#" @click.prevent="newItem">Nuevo Contacto</a>
      <search @search="searchItem"></search>
    </div>
    <contact-list
      :items="searchedItems"
      :query="query"
      @edit="editItem"
      @delete="deleteItem"
    ></contact-list>
    <contact-modal
      v-if="openModal"
      :edit-item="item"
      :countries="countries"
      @close="closeModal"
      @save="saveItem"
    ></contact-modal>
  </div>
</template>
<script>
import Search from "./Search";
import ContactIcon from "./ContactIcon";
import ContactList from "./ContactList";
import ContactModal from "./ContactModal";

import { getCountriesByRegion } from "../../services/Countries";
import { getContacts } from "../../services/Contacts";

export default {
  name: "contact-page",
  components: {
    Search,
    ContactIcon,
    ContactList,
    ContactModal
  },
  data() {
    return {
      query: "",
      openModal: false,
      items: [],
      item: {},
      itemId: 1,
      countries: []
    };
  },
  mounted() {
    getCountriesByRegion("usan").then(response => {
      this.countries = response.data.map(function(country) {
        return { name: country.nativeName, code: country.alpha3Code };
      });
    });

    getContacts().then(response => {
      this.items = response.data.map(item => {
        item.id = this.itemId;
        this.itemId++;
        return item;
      });
    });
  },
  computed: {
    searchedItems() {
      if (!this.query) {
        return this.items;
      }
      return this.items.filter(item => {
        for (let fieldName in item) {
          if (
            fieldName != "id" &&
            item[fieldName].toLowerCase().includes(this.query)
          ) {
            return true;
          }
        }
        return false;
      });
    }
  },
  methods: {
    newItem() {
      this.item = {};
      this.openModal = true;
    },
    editItem(item) {
      this.item = item;
      this.openModal = true;
    },
    deleteItem(index) {
      this.items.splice(index, 1);
    },
    searchItem(query) {
      this.query = query.toLowerCase();
    },
    saveItem(item) {
      if (item.id) {
        const index = this.items.findIndex(obj => obj.id == item.id);
        this.items.splice(index, 1, item);
      } else {
        item.id = this.itemId;
        this.items.push(item);
        this.itemId++;
      }

      this.closeModal();
    },
    closeModal() {
      this.openModal = false;
    }
  }
};
</script>

<style lang="scss" scoped>
.contact-page {
  margin: 10px;
  background-color: var(--clr-body);
  display: inline-block;
  border-radius: 5px;

  h3 {
    width: 35%;
    text-align: center;
    margin-top: 15px;
  }

  .menu {
    display: flex;
    justify-content: space-between;
    margin: 10px;
    align-items: flex-end;
  }
}
</style>
