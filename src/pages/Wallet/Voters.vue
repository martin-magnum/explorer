<template>
  <div class="max-w-2xl mx-auto md:pt-5 header-container">
    <ContentHeader
      >{{ $t("WALLET.DELEGATE.VOTERS") }}
        <span v-if="delegate">
          <UnikDisplay v-if="delegate.unikname" :unikname="delegate.unikname" :type="delegate.unikType" />
          <span v-else>
          {{ delegate.username }}
          </span>
        </span>
        </ContentHeader
    >
    <section class="page-section py-5 md:py-10">
      <div class="hidden sm:block">
        <TableWalletsDesktop
          :wallets="wallets"
          :total="delegate ? delegate.votes : 0"
          :sort-query="sortParams"
          @on-sort-change="onSortChange"
        />
      </div>
      <div class="sm:hidden">
        <TableWalletsMobile :wallets="wallets" :total="delegate ? delegate.votes : 0" />
      </div>
      <Pagination v-if="showPagination" :meta="meta" :current-page="currentPage" @page-change="onPageChange" />
    </section>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from "vue-property-decorator";
import { Route } from "vue-router";
import { IBlock, IDelegate, ISortParameters, IWallet } from "@/interfaces";
import DelegateService from "@/services/delegate";

Component.registerHooks(["beforeRouteEnter", "beforeRouteUpdate"]);

import UnikDisplay from "@/components/unik/UnikDisplay.vue";

@Component({
  components: { UnikDisplay },
})
export default class WalletVoters extends Vue {
  private delegate: IDelegate | null = null;
  private wallets: IWallet[] | null = null;
  private meta: any | null = null;
  private currentPage: number = 0;

  public static pageName: string = "wallet-voters"

  get showPagination() {
    return this.meta && this.meta.pageCount > 1;
  }

  get address() {
    return this.$route.params.address;
  }

  get sortParams() {
    return this.$store.getters["ui/walletSortParams"];
  }

  set sortParams(params: ISortParameters) {
    this.$store.dispatch("ui/setWalletSortParams", {
      field: params.field,
      type: params.type,
    });
  }

  @Watch("currentPage")
  public onCurrentPageChanged() {
    this.changePage();
  }

  public async beforeRouteEnter(to: Route, from: Route, next: (vm: any) => void) {
    try {
      const delegate = await DelegateService.find(to.params.address, true);
      const { meta, data } = await DelegateService.voters(to.params.address, Number(to.params.page));

      next((vm: WalletVoters) => {
        vm.currentPage = Number(to.params.page);
        vm.setDelegate(delegate);
        // @ts-ignore
        vm.setWallets(data);
        vm.setMeta(meta);
      });
    } catch (e) {
      next({ name: "404" });
    }
  }

  public async beforeRouteUpdate(to: Route, from: Route, next: (vm?: any) => void) {
    this.wallets = null;
    this.meta = null;

    try {
      const delegate = await DelegateService.find(to.params.address, true);
      const { meta, data } = await DelegateService.voters(to.params.address, Number(to.params.page));

      this.currentPage = Number(to.params.page);
      this.setDelegate(delegate);
      // @ts-ignore
      this.setWallets(data);
      this.setMeta(meta);
      next();
    } catch (e) {
      next({ name: "404" });
    }
  }

  private setDelegate(delegate: IDelegate) {
    this.delegate = delegate;
  }

  private setWallets(wallets: IWallet[]) {
    this.wallets = wallets;
  }

  private setMeta(meta: any) {
    this.meta = meta;
  }

  private onPageChange(page: number) {
    this.currentPage = page;
  }

  private changePage() {
    if (this.currentPage !== Number(this.$route.params.page) || this.address !== this.$route.params.address) {
      // @ts-ignore
      this.$router.push({
        name: WalletVoters.pageName,
        params: {
          address: this.address,
          page: this.currentPage,
        },
      });
    }
  }

  private onSortChange(params: ISortParameters) {
    this.sortParams = params;
  }
}
</script>

<style>
.header-container h1 {
  display: flex;
  flex-direction: row;
}

.header-container h1 .unik svg {
  font-size: 1.1em;
  margin-left: 0.25em;
  border-radius: 4px;
}

</style>
