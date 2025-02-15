<template>
  <div class="field">
    <label class="label">Mint authority*</label>
    <secret-form-field
      v-model:signExternally="mintAuthoritySignsExternally"
      v-model:secret="mintAuthoritySecret"
    />
  </div>
  <div class="field">
    <label class="label">Destination account*</label>
    <public-key-form-field v-model:address="destinationAccountAddress" />
  </div>
  <div class="field">
    <label class="label">Amount*</label>
    <div class="control">
      <input
        v-model="tokenAmount"
        class="input is-black"
        type="text"
        placeholder="Amount of tokens to mint e.g. 20000"
      />
    </div>
    <p class="help">
      Please be aware that a token is minted using its smallest denomination
      e.g. if you have a token with 2 decimals and you type in 200 you will mint
      2 tokens.
    </p>
  </div>
  <div style="display: flex" class="control is-justify-content-center mt-5">
    <button
      :class="{ 'is-loading': mintingToAccount }"
      class="button is-black"
      @click="mintToAccount"
    >
      Mint to account
    </button>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, toRefs } from "vue";
import { mintToken } from "@/solana/token";
import accountComponents from "../accountComponents";
import { u64 } from "@solana/spl-token";
import SecretFormField from "@/components/util/SecretFormField.vue";
import PublicKeyFormField from "@/components/util/PublicKeyFormField.vue";

export default defineComponent({
  name: accountComponents.Mint,
  components: {
    SecretFormField,
    PublicKeyFormField,
  },
  emits: ["update:accountAddress"],
  props: {
    payerSecret: {
      type: String,
      required: true,
    },
    payerSignsExternally: {
      type: Boolean,
      default: true,
    },
  },
  setup(props, { emit }) {
    const { payerSecret, payerSignsExternally } = toRefs(props);
    const mintAuthoritySecret = ref("");
    const mintAuthoritySignsExternally = ref(true);

    const destinationAccountAddress = ref("");
    const mintingToAccount = ref(false);
    const tokenAmount = ref("");

    const mintToAccount = async () => {
      mintingToAccount.value = true;
      emit("update:accountAddress", "");
      try {
        await mintToken(
          payerSecret.value,
          mintAuthoritySecret.value,
          destinationAccountAddress.value,
          new u64(tokenAmount.value, 10),
          payerSignsExternally.value,
          mintAuthoritySignsExternally.value
        );
        emit("update:accountAddress", destinationAccountAddress.value);
      } catch (err) {
        mintingToAccount.value = false;
        throw err;
      }
      mintingToAccount.value = false;
    };

    return {
      destinationAccountAddress,
      mintingToAccount,
      mintToAccount,
      mintAuthoritySecret,
      tokenAmount,
      mintAuthoritySignsExternally,
    };
  },
});
</script>
