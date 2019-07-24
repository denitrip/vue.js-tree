<template>
  <li>
    <div>
      <input type="checkbox" v-model="isSelected" @change="setChildrenSelection"
             :indeterminate.prop="isIndeterminateState || childrensIndeterminateState">
      <label @click="toggle">{{ item.name }}</label>
      <span v-if="isBranch">({{ isOpen ? '-' : '+' }})</span>
    </div>
    <ul v-show="isOpen" v-if="isBranch">
      <tree-item
        ref="child"
        v-for="(child, index) in item.children"
        :key="index"
        :item="child"
        :isParentSelected="isSelected"
        :isParentOpen="isOpen"
        :isSingleSelectedParent="isSingleSelectedItem"
        @countSelectedChildrens="countSelectedChildrens"
        @checkboxStatusIndeterminate="checkboxStatusIndeterminate"
      ></tree-item>
    </ul>
  </li>
</template>

<script>
  export default {
    name: "TreeItem",
    props: ['item', 'isParentSelected', 'isParentOpen'],
    data() {
      return {
        isOpen: false,
        isSelected: false,
        childrensSelected: 0,
        childrensIndeterminateState: false,
        isSingleSelectedItem: false
      }
    },
    computed: {
      isBranch() {
        return this.item.children && this.item.children.length
      },
      numOfChildrens() {
        return this.item.children ? this.item.children.length : 0;
      },
      isIndeterminateState() {
        return (this.childrensSelected > 0 && this.childrensSelected < this.numOfChildrens);
      },
      isNotIndeterminateWithSelections() {
        return this.numOfChildrens && !this.isSelected && this.numOfChildrens === this.childrensSelected && !this.isIndeterminateState && !this.childrensIndeterminateState;
      },
      nestingState() {
        return this.childrensIndeterminateState || this.isNotIndeterminateWithSelections || this.isIndeterminateState;
      }
    },
    methods: {
      test() {
        this.isSelected = !this.isSelected;
      },
      toggle() {
        if (this.isBranch) {
          this.isOpen = !this.isOpen
        }
      },
      setChildrenSelection(e) {
        if (this.nestingState) {
          this.setSelection(false);
          this.isSelected = e.target.checked;
        }
        else {
          this.setSelection(true);
        }
        this.childrensSelected = this.isSelected ? this.numOfChildrens : 0;
        this.$emit('countSelectedChildrens', this.isSelected);
      },
      countSelectedChildrens(state) {
        if (state && this.childrensSelected < this.numOfChildrens) {
          this.childrensSelected++;
        }
        else if (this.childrensSelected > 0) {
          this.childrensSelected--;
        }
      },
      checkboxStatusIndeterminate(state) {
        if (this.$refs.child) {
          this.childrensIndeterminateState = !!this.$refs.child.find(e => e.isIndeterminateState || e.childrensIndeterminateState || e.isNotIndeterminateWithSelections);
        } else {
          this.childrensIndeterminateState = state;
        }
      },
      setSelection(parentState) {
        if (parentState) this.childrensSelected = this.numOfChildrens;
        else this.childrensSelected = 0;
        if (this.$refs.child) {
          this.$refs.child.forEach(e => {
            if (e.numOfChildrens) e.setSelection(parentState);
            else e.isSelected = parentState
          })
        }
        this.isSelected = parentState;
      }
    },
    watch: {
      isParentOpen: function (parentState) {
        this.isOpen = parentState ? parentState : false;
      },
      isIndeterminateState: function (state) {
        this.$emit('checkboxStatusIndeterminate', state);
      },
      childrensIndeterminateState: function (state) {
        this.$emit('checkboxStatusIndeterminate', state);
      },
      isNotIndeterminateWithSelections: function (state) {
        this.$emit('checkboxStatusIndeterminate', state);
      }
    }
  }
</script>

<style scoped>

  li {
    list-style-type: none;
  }

  ul {
    margin-block-start: 0;
  }

</style>
