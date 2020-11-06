<template>
  <a-form :model="form" :label-col="labelCol" :wrapper-col="wrapperCol">
    <template v-for="ty_arg in tyArgs" :key="ty_arg">
      <a-form-item>
        <template v-slot:label>
          TypeArg: {{ ty_arg.name }}
        </template>
        <a-input v-model:value="form.ty_args[ty_arg.name]" />
      </a-form-item>
    </template>

    <template v-for="arg in args" :key="arg">
      <a-form-item>
        <template v-slot:label>
          Arg: {{ arg.name }}
        </template>
        <a-input v-model:value="form.args[arg.name]" :placeholder="formatTypeTag(arg.type_tag)" />
      </a-form-item>
    </template>

    <a-form-item :wrapper-col="{ span: 14, offset: 4 }">
      <a-button type="primary" @click="onSubmit">
        Create
      </a-button>
      <a-button style="margin-left: 10px;" @click="onCancel">
        Cancel
      </a-button>
    </a-form-item>
  </a-form>
</template>
<script lang="ts">
import { Options, Vue } from 'vue-class-component';
import { starcoin_types, types } from 'starcoin-typescript-sdk';
import { encode_tx_payload } from 'starcoin-typescript-sdk/build/main/utils/tx';
import { address_from_json } from 'starcoin-typescript-sdk/build/main/utils/lcs_to_json';
import { fromHexString } from 'starcoin-typescript-sdk/build/main/utils/hex';
import { inject } from 'vue';

@Options({
  props: {
    tyArgs: Array,
    args: Array,
    code: Object,
  }
})
export default class ScriptForm extends Vue {
  tyArgs!: types.TypeArgumentABI[]
  args!: types.ArgumentABI[]
  code!: Uint8Array

  form!: any
  data() {
    return {
      labelCol: { span: 4 },
      wrapperCol: { span: 14 },
      form: {
        ty_args: {},
        args: {},
        region: undefined,
        date1: undefined,
        delivery: false,
        type: [],
        resource: '',
        desc: '',
      },
    };
  }

  onSubmit() {
    console.log('submit!', JSON.stringify(this.form));

    const typeTags = this.tyArgs.map(a => {
      let name = this.form.ty_args[a.name] as string;
      let parts = name.split("::", 3);
      let addr = address_from_json(parts[0]);
      const s = new starcoin_types.StructTag(addr, new starcoin_types.Identifier(parts[1]), new starcoin_types.Identifier(parts[2]), []);
      return new starcoin_types.TypeTagVariantStruct(s);
    });
    const scriptArgs = this.args.map(a => {
      let value = this.form.args[a.name] as string;
      let valueType = a.type_tag;
      return this.encodeTransactionArgument(valueType, value || "");
    });
    const payload = encode_tx_payload(this.code, typeTags, scriptArgs);
    const relocate = `stc://wallet.starcoin.org/${payload}`;
    window.location.replace(relocate);
  }

  onCancel() {
  }

  encodeTransactionArgument(ty: types.TypeTag, value: string): starcoin_types.TransactionArgument {
    if (ty === 'Bool') {
        return new starcoin_types.TransactionArgumentVariantBool(value === 'true');
    } else if (ty === 'U8') {
      let n = Number.parseInt(value);
      return new starcoin_types.TransactionArgumentVariantU8(n);
    }else if (ty === 'U64') {
      return new starcoin_types.TransactionArgumentVariantU64(BigInt(value));
    }else if (ty === 'U128') {
      let n = BigInt(value);
      return new starcoin_types.TransactionArgumentVariantU128(n);
    } else if (ty === 'Address') {
      let addr = address_from_json(value);
      return new starcoin_types.TransactionArgumentVariantAddress(addr);
    } else {
      let bytes = fromHexString(value || "");
      return new starcoin_types.TransactionArgumentVariantU8Vector(bytes);
    }
  }

  formatTypeTag(typeTag: types.TypeTag): string {
    if(typeof typeTag === 'string') {
      return typeTag;
    } else if ('Vector' in typeTag) {
      return `Vector<${this.formatTypeTag(typeTag.Vector)}>`
    } else if ('Struct' in typeTag) {
      let st = typeTag.Struct;
      let base = `${st.address}::${st.module}::${st.name}`;

      if( st.type_params.length === 0) {
        return base;
      }
      const s= st.type_params.map(t => this.formatTypeTag(t)).join(",");
      return base + '<' + s + '>';
    } else {
      return typeTag;
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
