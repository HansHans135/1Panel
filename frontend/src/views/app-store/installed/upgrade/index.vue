<template>
    <el-drawer :close-on-click-modal="false" v-model="open" size="30%">
        <template #header>
            <Header :header="$t('app.upgrade')" :resource="resourceName" :back="handleClose"></Header>
        </template>
        <el-row>
            <el-col :span="22" :offset="1">
                <el-form
                    @submit.prevent
                    ref="updateRef"
                    :rules="rules"
                    label-position="top"
                    :model="operateReq"
                    v-loading="loading"
                >
                    <el-form-item :label="$t('app.versionSelect')" prop="detailId">
                        <el-select v-model="operateReq.detailId">
                            <el-option
                                v-for="(version, index) in versions"
                                :key="index"
                                :value="version.detailId"
                                :label="version.version"
                            ></el-option>
                        </el-select>
                    </el-form-item>
                </el-form>
            </el-col>
        </el-row>
        <template #footer>
            <span class="dialog-footer">
                <el-button @click="handleClose" :disabled="loading">{{ $t('commons.button.cancel') }}</el-button>
                <el-button type="primary" @click="onOperate" :disabled="versions == null || loading">
                    {{ $t('commons.button.confirm') }}
                </el-button>
            </span>
        </template>
    </el-drawer>
</template>
<script lang="ts" setup>
import { App } from '@/api/interface/app';
import { GetAppUpdateVersions, InstalledOp } from '@/api/modules/app';
import i18n from '@/lang';
import { ElMessageBox, FormInstance } from 'element-plus';
import { reactive, ref, onBeforeUnmount } from 'vue';
import Header from '@/components/drawer-header/index.vue';
import { MsgSuccess } from '@/utils/message';
import { Rules } from '@/global/form-rules';
import bus from '../../bus';

const updateRef = ref<FormInstance>();
let open = ref(false);
let loading = ref(false);
let versions = ref<App.VersionDetail[]>();
let operateReq = reactive({
    detailId: 0,
    operate: 'upgrade',
    installId: 0,
});
const resourceName = ref('');
let rules = ref<any>({
    detailId: [Rules.requiredSelect],
});

const em = defineEmits(['close']);
const handleClose = () => {
    open.value = false;
    em('close', open);
};

const acceptParams = (id: number, name: string) => {
    operateReq.installId = id;
    resourceName.value = name;
    GetAppUpdateVersions(id).then((res) => {
        versions.value = res.data;
        if (res.data != null && res.data.length > 0) {
            operateReq.detailId = res.data[0].detailId;
        }
        open.value = true;
    });
};

const operate = async () => {
    loading.value = true;
    await InstalledOp(operateReq)
        .then(() => {
            MsgSuccess(i18n.global.t('app.upgradeStart'));
            bus.emit('upgrade', true);
            handleClose();
        })
        .finally(() => {
            loading.value = false;
        });
};

const onOperate = async () => {
    ElMessageBox.confirm(
        i18n.global.t('app.operatorHelper', [i18n.global.t('app.upgrade')]),
        i18n.global.t('app.upgrade'),
        {
            confirmButtonText: i18n.global.t('commons.button.confirm'),
            cancelButtonText: i18n.global.t('commons.button.cancel'),
            type: 'info',
        },
    ).then(() => {
        operate();
    });
};

onBeforeUnmount(() => {
    bus.off('upgrade');
});

defineExpose({
    acceptParams,
});
</script>
