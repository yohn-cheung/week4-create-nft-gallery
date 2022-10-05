<script setup>
import { ref } from "vue";

const wallet = ref("");
const collection = ref("");
const checked = ref(false);
const NFTs = ref(null);
const pageKeys = ref([]);

const currentPage = ref(0);
const next = ref(false);
const message = ref("");
const show = ref(false);

const api_key = import.meta.env.VITE_MY_API_KEY;

function startSearching() {
  show.value = false;
  if (wallet.value === "" && collection.value === "") return;
  if (checked.value) {
    fetchNFTsForCollection();
  } else {
    fetchNFTs();
  }
}

async function fetchNFTsForCollection() {
  NFTs.value = null;
  if (collection.value.length) {
    var requestOptions = {
      method: "GET",
    };
    // const api_key = "";
    const baseURL = `https://eth-mainnet.alchemyapi.io/v2/${api_key}/getNFTsForCollection/`;
    const fetchURL = `${baseURL}?contractAddress=${
      collection.value
    }&withMetadata=${"true"}`;
    const nfts = await fetch(fetchURL, requestOptions)
      .then((data) => data.json())
      .catch((error) => {
        show.value = true;
        message.value = "ENTER A VALID ADDRESS";
      });

    if (nfts) {
      NFTs.value = nfts.nfts;
    } else {
      console.log("no NFTS");
      show.value = true;
      message.value = "NO NFT FOUND, TRY A NEW ADDRESS";
    }
  }
}

async function fetchNFTs() {
  NFTs.value = null;
  next.value = false;
  let nfts;
  // const api_key = "";
  const baseURL = `https://eth-mainnet.alchemyapi.io/v2/${api_key}/getNFTs/`;
  var requestOptions = {
    method: "GET",
  };

  if (!collection.value.length) {
    let fetchURL;
    if (pageKeys.value.length > 0) {
      fetchURL = `${baseURL}?owner=${wallet.value}&pageSize=9&pageKey=${
        pageKeys.value[currentPage.value - 1]
      }`;
    } else {
      fetchURL = `${baseURL}?owner=${wallet.value}&pageSize=9`;
    }

    nfts = await fetch(fetchURL, requestOptions)
      .then((data) => data.json())
      .catch(() => {
        show.value = true;
        message.value = "ENTER A VALID ADDRESS";
      });
  } else {
    console.log("fetching nfts for collection owned by address");
    const fetchURL = `${baseURL}?owner=${wallet.value}&contractAddresses%5B%5D=${collection.value}&pageSize=9`;
    nfts = await fetch(fetchURL, requestOptions)
      .then((data) => data.json())
      .catch(() => {
        show.value = true;
        message.value = "ENTER A VALID ADDRESS";
      });
  }

  if (nfts.totalCount !== 0) {
    NFTs.value = nfts.ownedNfts;
    show.value = false;
  } else {
    show.value = true;
    message.value = "NO NFT FOUND, TRY A NEW ADDRESS";
  }

  if (nfts.hasOwnProperty("pageKey")) {
    if (!pageKeys.value.includes(nfts.pageKey)) {
      pageKeys.value.push(nfts.pageKey);
    }
  } else {
    next.value = true;
  }
}

function copyAddress(address) {
  navigator.clipboard.writeText(address).then(() => {
    // Alert the user that the action took place.
    // Nobody likes hidden stuff being done under the hood!
    alert("Copied");
  });
}

function onClickPage(pageIndex) {
  currentPage.value = pageIndex;
  startSearching();
}
</script>

<template>
  <div v-if="show" class="alert alert-danger" role="alert">
    {{ message }}
  </div>

  <div class="container">
    <h2 class="text-center mt-3 mb-5">Search for NFT's</h2>
    <div class="input-group mb-3">
      <input
        v-model="wallet"
        placeholder="Add your wallet address"
        class="form-control"
        :disabled="checked"
      />
    </div>

    <div class="input-group mb-3">
      <input
        v-model="collection"
        placeholder="Add the collection address"
        class="form-control"
      />
    </div>
    <div class="input-group mb-3">
      <div class="input-group-text">
        <input
          v-model="checked"
          class="form-check-input mt-0"
          type="checkbox"
          value=""
          aria-label="Checkbox for following text input"
        />

        Fetch for collection
      </div>
    </div>

    <div class="d-grid gap-2">
      <button class="btn btn-primary" type="button" @click="startSearching()">
        Start searching
      </button>
    </div>

    <section class="row g-2 mt-3">
      <div class="col-4" v-for="(nft, index) in NFTs" :key="index">
        <div class="card">
          <img
            :src="nft.media[0].gateway"
            class="card-img-top"
            :alt="nft.title"
          />
          <div class="card-body">
            <h5 class="card-title">{{ nft.title }}</h5>
            <h6 class="card-subtitle mb-2 text-muted">
              Id: {{ nft.id.tokenId.substr(nft.id.tokenId.length - 4) }}
            </h6>
            <h6 class="card-subtitle mb-2 text-muted">
              <i @click="copyAddress(nft.contract.address)">
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  width="16"
                  height="16"
                  fill="currentColor"
                  class="bi bi-clipboard"
                  viewBox="0 0 16 16"
                >
                  <path
                    d="M4 1.5H3a2 2 0 0 0-2 2V14a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V3.5a2 2 0 0 0-2-2h-1v1h1a1 1 0 0 1 1 1V14a1 1 0 0 1-1 1H3a1 1 0 0 1-1-1V3.5a1 1 0 0 1 1-1h1v-1z"
                  />
                  <path
                    d="M9.5 1a.5.5 0 0 1 .5.5v1a.5.5 0 0 1-.5.5h-3a.5.5 0 0 1-.5-.5v-1a.5.5 0 0 1 .5-.5h3zm-3-1A1.5 1.5 0 0 0 5 1.5v1A1.5 1.5 0 0 0 6.5 4h3A1.5 1.5 0 0 0 11 2.5v-1A1.5 1.5 0 0 0 9.5 0h-3z"
                  />
                </svg>
              </i>
              {{
                `${nft.contract.address.substr(
                  0,
                  4
                )}...${nft.contract.address.substr(
                  nft.contract.address.length - 4
                )}`
              }}
            </h6>
            <p class="card-text">
              {{ nft.description?.substr(0, 150) }}
            </p>
            <a
              :href="`https://etherscan.io/token/${nft.contract.address}`"
              target="_blank"
              class="btn btn-primary"
              >View on etherscan</a
            >
          </div>
        </div>
      </div>
    </section>

    <section v-if="pageKeys.length > 0">
      <nav aria-label="navigation">
        <ul class="pagination justify-content-center">
          <li class="page-item" :class="{ disabled: currentPage === 0 }">
            <i
              @click="onClickPage(currentPage - 1)"
              class="page-link"
              aria-label="Previous"
            >
              <span aria-hidden="true">&laquo;</span>
            </i>
          </li>
          <li class="page-item" :class="{ disabled: next }">
            <i
              @click="onClickPage(currentPage + 1)"
              class="page-link"
              aria-label="Next"
            >
              <span aria-hidden="true">&raquo;</span>
            </i>
          </li>
        </ul>
      </nav>
    </section>
  </div>
</template>
