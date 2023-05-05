<script>
  import { createEventDispatcher } from "svelte";
  import Modal from "./Modal.svelte";

  let showModal = false;

  const dispatch = createEventDispatcher();

  export let score;
  export let genre;
  export let tags;
  export let search;

  const debounce = (func, delay) => {
    let timer;
    return (...args) => {
      clearTimeout(timer);
      timer = setTimeout(() => {
        func(...args);
      }, delay);
    };
  };

  const handleInput = debounce((event) => {
    const value = event.target.value.trim();
    let tagArray = [];
    if (typeof value === "string") {
      tagArray = value.split(", ");
    }
    score = value;
    genre = value;
    tags = tagArray;
    search = value;
    dispatch("scoreChange", { score });
    dispatch("genreChange", { genre });
    dispatch("tagsChange", { tags });
    dispatch("searchChange", { search });
  }, 2000);
</script>

<div class="flex space-x-2">
  <input
    type="text"
    class="border w-full text-black rounded-sm focus:outline-none hover:cursor-pointer"
    on:input={handleInput}
  />
  <a href="/" type="button" class="bg-white text-black px-2 flex items-center">
    <svg
      height="18"
      width="18"
      fill="#000"
      stroke="currentColor"
      stroke-width="1.5"
      viewBox="0 0 24 24"
      xmlns="http://www.w3.org/2000/svg"
      aria-hidden="true"
    >
      <path
        stroke-linecap="round"
        stroke-linejoin="round"
        d="M10.5 6h9.75M10.5 6a1.5 1.5 0 11-3 0m3 0a1.5 1.5 0 10-3 0M3.75 6H7.5m3 12h9.75m-9.75 0a1.5 1.5 0 01-3 0m3 0a1.5 1.5 0 00-3 0m-3.75 0H7.5m9-6h3.75m-3.75 0a1.5 1.5 0 01-3 0m3 0a1.5 1.5 0 00-3 0m-9.75 0h9.75"
      />
    </svg>
  </a>
  <!-- <button on:click={() => (showModal = true)}> show modal </button> -->
</div>
<Modal bind:showModal />
