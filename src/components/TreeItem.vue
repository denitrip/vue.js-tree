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
        :isParentOpen="isOpen"
        :isParentSelected="isSelected"
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
    props: ['item', 'isParentOpen', 'isParentSelected'],
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
        return (this.numOfChildrens && !this.isSelected && this.numOfChildrens === this.childrensSelected && !this.isIndeterminateState && !this.childrensIndeterminateState)
      },
      nestingState() {
        return this.childrensIndeterminateState || this.isNotIndeterminateWithSelections || this.isIndeterminateState
      },
      isNoSelectionsInside() {
        return this.isSelected && this.isParentSelected && this.numOfChildrens && !this.childrensSelected
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
        if (this.nestingState || (this.isSelected && this.numOfChildrens === this.childrensSelected)) {
          this.setSelection(false);
          this.isSelected = e.target.checked;
        }
        else {
          this.setSelection(true);
          this.isSelected = e.target.checked;
        }
        this.childrensSelected = this.isSelected ? this.numOfChildrens : 0;
        this.$emit('countSelectedChildrens', e.target.checked);
      },
      countSelectedChildrens(state) {
        if (state && this.childrensSelected < this.numOfChildrens) {
          this.childrensSelected++;
        }
        else if (!state && this.childrensSelected > 0) {
          this.childrensSelected--;
        }
      },
      checkboxStatusIndeterminate(state) {
        if (this.$refs.child) {
          this.childrensIndeterminateState = !!this.$refs.child.find(e => e.isIndeterminateState || e.childrensIndeterminateState || e.isNotIndeterminateWithSelections || (e.isSelected && e.isNoSelectionsInside));
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
      },
      isNoSelectionsInside: function (state) {
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
