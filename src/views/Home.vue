<template>
  <a-upload
    v-model:fileList="fileList"
    accept=".abi"
    :multiple="true"
    :showUploadList="false"
    :beforeUpload="beforeUpload"
    :customRequest="customRequest"
    :remove="remove"
  >
    <a-button>
      <upload-outlined />
      导入 ABI
    </a-button>
  </a-upload>

  <a-list item-layout="horizontal" :data-source="scriptList">
    <template v-slot:renderItem="{  item }">
      <a-list-item>
        <template v-slot:actions>
          <h2 v-if="ongoing[item.name]">
            hahah
          </h2>
          <a-button v-else @click="onClick" :id="item.name">发起</a-button>
        </template>
        <a-list-item-meta>
          <template v-slot:title>
            {{ item.name }}
          </template>
          <template v-slot:description>

          </template>

        </a-list-item-meta>
      </a-list-item>
    </template>
  </a-list>
  <template v-if="current">
    <ScriptForm
      :ty-args="current.ty_args"
      :args="current.args"
      :code="current.code">
    </ScriptForm>
  </template>
</template>

<script lang="ts">
// import { message } from 'ant-design-vue';

import { UploadOutlined } from '@ant-design/icons-vue';
import { abi } from 'starcoin-typescript-sdk';
import { types } from 'starcoin-typescript-sdk';
import {Options, Vue } from 'vue-class-component';
import ScriptForm from '@/components/ScriptForm.vue';
import {provide, ref} from 'vue';

@Options({
  components: {UploadOutlined, ScriptForm }
})
export default class Home extends Vue {
  scripts: {[K in types.ScriptABI['name']]: types.ScriptABI} = {};
  fileList: any[] = [];

  ongoing: {[K in string]: boolean} = {};
  current: types.ScriptABI | null = null;
  get scriptList() {
    return Object.values(this.scripts);
  }

  formatABI(abi: types.ScriptABI): string {
    // let ty_args = '';
    // if (abi.ty_args.length > 0) {
    //   const tys = abi.ty_args.join(', ');
    //   ty_args = `<${tys}>`;
    // }
    return abi.args.map(arg => {
      arg.type_tag;
    }).join(',');
  }
  onClick(event: any) {
    const abi_id = event.target.id;
    // this.ongoing[abi_id] = true;
    let abi = this.scripts[abi_id as string];
    console.log(abi);
    this.current = abi;

    // const payload = encode_tx_payload(abi.code, [], []);
    //
    // const relocate = `stc://wallet.starcoin.org/${payload}`;
    // window.location.replace(relocate);
  }

  remove(file: any) {
    console.log('remove', file);
    return true;
  }
  customRequest(args: any) {
    console.log('customRequest', args);
  }
  async beforeUpload(file: File, fileList: FileList) {
    console.log('beforeUpload', file, fileList);
    // this.fileList.push(file);
    console.log('this.fileList', this.fileList.length);
    const buffer = await file.arrayBuffer();
    const scriptABI = abi.decodeABI(new Uint8Array(buffer));
    this.scripts[scriptABI.name] = scriptABI;
    return false;
  }
}

// export default defineComponent({
//   components: {
//     UploadOutlined
//   },
//   data() {
//     return {
//       fileList: [],
//       scripts: []
//     };
//   },
//   methods: {
//     formatABI(abi: types.ScriptABI): string {
//       // let ty_args = '';
//       // if (abi.ty_args.length > 0) {
//       //   const tys = abi.ty_args.join(', ');
//       //   ty_args = `<${tys}>`;
//       // }
//       return abi.args.map(arg => {
//         arg.type_tag;
//       }).join(',');
//     }
//     ,
//     remove(file: any) {
//       console.log('remove', file);
//       return true;
//     },
//     customRequest(args: any) {
//       console.log('customRequest', args);
//     },
//     async beforeUpload(file: File, fileList: FileList) {
//       console.log('beforeUpload', file, fileList);
//       // this.fileList.push(file);
//       console.log('this.fileList', this.fileList.length);
//       const buffer = await file.arrayBuffer();
//       const scriptABI = abi.decodeABI(new Uint8Array(buffer));
//       this.scripts.push(scriptABI);
//       console.log(scriptABI);
//       return false;
//     }
//   }
// });
// function formatTypeTag(t: TypeTag): string {
//   if ('Vector' in t) {
//     `Vector<${formatTypeTag(t.Vector)}>`
//       }
//   else if ('Struct' in t) {
//     let structTag = t.Struct;
//     structTag
//   }
// }
</script>

