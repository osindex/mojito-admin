<template>
  <custom-scroll-drawer :title="actionType === 'add' ? $t('add') : $t('edit') " v-model="drawer" direction="rtl" :size="500">
    <el-form :model="form" :rules="rules" ref="formRef" label-width="110px" size="small">
      <el-form-item :label="$t('name')" prop="name">
        <el-input v-model="form.name"></el-input>
      </el-form-item>
      <el-form-item :label="$t('router')" prop="uri">
        <el-input v-model="form.uri"></el-input>
      </el-form-item>
      <el-form-item :label="$t('guardName')" prop="guard_name">
        <guard-select v-model="form.guard_name"></guard-select>
      </el-form-item>
      <el-form-item :label="$t('parentMenu')" prop="parent_id">
        <menu-cascader v-model="form.parent_id" :guard-name="form.guard_name"></menu-cascader>
      </el-form-item>
      <el-form-item :label="$t('permission')" prop="permission_name">
        <el-input v-model="form.permission_name"></el-input>
      </el-form-item>
      <el-form-item :label="$t('icon')" prop="icon">
        <el-input v-model="form.icon"></el-input>
      </el-form-item>
      <el-form-item :label="$t('sequence')" prop="sequence">
        <el-input v-model.number="form.sequence"></el-input>
      </el-form-item>
    </el-form>
    <template #footer>
      <el-button @click="dialogformVisible = false" size="small">{{ $t('cancel') }}</el-button>
      <el-button type="primary" @click="handleAdd" v-if="actionType === 'add'" size="small">{{ $t('confirm') }}</el-button>
      <el-button type="primary" @click="handleEdit" v-if="actionType === 'edit'" size="small">{{ $t('confirm') }}</el-button>
    </template>
  </custom-scroll-drawer>
</template>
<script>
import CustomScrollDrawer from '@/components/Drawer/CustomScrollDrawer.vue'
import GuardSelect from '@/components/Select/GuardSelect.vue'
import PermissionGroupSelect from '@/components/Select/PermissionGroupSelect.vue'
import MenuCascader from '@/components/Cascader/MenuCascader.vue'
import { defineComponent, ref, watch } from 'vue'
import { addMenu, editMenu } from '@/api/menu'
import notice from '@/utils/notice'

export default defineComponent({
  components: { 
    CustomScrollDrawer,
    GuardSelect,
    PermissionGroupSelect,
    MenuCascader,
  },
  name: 'PermissionFormDrawer',
  props: {
    modelValue: false,
    title: {
      type: String,
    },
    action: {
      type: String,
      default: 'add'
    },
    row: {
      type: Object,
    }
  },
  setup(props, { emit }) {
    const drawer = ref(false)
    const actionType = ref(props.action)

    let defualtForm = {
      name: null,
      uri: null,
      permission_name: null,
      guard_name: null,
      parent_id: null,
      icon: null,
      sequence: null,
    }
    const form = ref(defualtForm)
    const formRef = ref(null)

    watch(() => props.modelValue, (v) => {
      drawer.value = v
    })

    watch(() => props.action, (a) => {
      if (a === 'add') {
        form.value = defualtForm
      }
      actionType.value = a
    })

    watch(drawer, (d) => {
      emit("update:modelValue", d)
    })

    watch(() => props.row, (row) => {
      form.value = {
        name: row.name,
        uri: row.uri,
        permission_name: row.permission_name,
        guard_name: row.guard_name,
        parent_id: row.parent_id,
        icon: row.icon,
        sequence: row.sequence,
      }
    })

    const handleAdd = () => {
      formRef.value.validate((valid) => {
        if (!valid) {
          return false
        }
        addMenu(form.value).then(() => {
          notice.addSuccess()
          drawer.value = false
          formRef.value.resetFields()
        })
      })
    }

    const handleEdit = () => {
      formRef.value.validate((valid) => {
        if (!valid) {
          return false
        }
        editMenu(props.row.id, form.value).then(() => {
          notice.editSuccess()
          drawer.value = false
          props.row.name = form.value.name
          props.row.uri = form.value.uri
          props.row.permission_name = form.value.permission_name
          props.row.guard_name = form.value.guard_name
          props.row.parent_id = form.value.parent_id
          props.row.icon = form.value.icon
          props.row.sequence = form.value.sequence
          form.value = defualtForm
        })
      })
    }

    return {
      drawer,
      form,
      formRef,
      handleAdd,
      handleEdit,
      actionType,
      rules: {
        name: [
          { required: true },
          { min: 1, max: 255 }
        ],
        uri: [
          { required: true },
          { min: 1, max: 255 }
        ],
        guard_name: [
          { required: true },
          { min: 1, max: 255 }
        ],
        parent_id: [
          { type: 'number' }
        ],
        sequence: [
          { type: 'number' }
        ]
      }
    }
  },
})
</script>
<style lang="scss" scoped>

</style>