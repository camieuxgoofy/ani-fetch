<script>
  import { onMount, onDestroy } from "svelte";
  import { lazyLoad } from "./lazyLoad";
  import SearchBar from "./components/SearchBar.svelte";

  let aniData = [];
  const query = `
      query ($search: String ,$page: Int, $score: Int, $genre: String, $tags: [String!]) {
        Page (page: $page, perPage: 20) {
          pageInfo {
            currentPage
            lastPage
          },
          media (search: $search ,tag_in: $tags, genre: $genre ,averageScore_greater: $score) {
            id,
            title {
              romaji,
              english,
              native
            },
            episodes,
            genres,
            coverImage {
              medium
            },
            isAdult,
            averageScore,
            meanScore,
            tags {
              name
            },
            format
          }
        }
      }`;

  let score;
  let genre;
  let tags;
  let search;
  let page = 1;
  let totalPages = 1;
  let loading = false;
  let error = null;

  const variables = { page, score, genre, tags, search };

  const fetchResults = async () => {
    loading = true;

    const url = "https://graphql.anilist.co";
    const options = {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Accept: "application/json",
      },
      body: JSON.stringify({
        query: query,
        variables: variables,
      }),
    };
    try {
      const response = await fetch(url, options);
      const data = await response.json();
      aniData = data.data.Page.media;
      page = data.data.Page.pageInfo.currentPage;
      totalPages = data.data.Page.pageInfo.lastPage;
      console.log("total pages:", totalPages);
    } catch (error) {
      console.error("Error fetching data: ", error);
    } finally {
      loading = false;
    }
  };

  onMount(() => {
    fetchResults();
  });

  const nextPage = () => {
    if (page < totalPages) {
      variables.page = page + 1;
      fetchResults();
    }
  };

  const prevPage = () => {
    if (page > 1) {
      variables.page = page - 1;
      fetchResults();
    }
  };

  const handleSearch = (event) => {
    let newSearch = event.detail.search;
    let newScore = event.detail.score;
    let newGenre = event.detail.genre;
    let newTags = event.detail.tags;

    if (!Number.isInteger(newScore)) {
      newScore = 0;
    }

    if (newGenre == "") {
      newGenre = "";
    }

    search = newSearch;
    score = newScore;
    genre = newGenre;
    tags = newTags;
    variables.search = search;
    variables.genre = genre;
    variables.score = score;
    variables.tags = tags;
    fetchResults();
  };
</script>

<section class="space-y-4">
  <SearchBar
    {search}
    {score}
    {genre}
    {tags}
    on:searchChange={handleSearch}
    on:scoreChange={handleSearch}
    on:genreChange={handleSearch}
    on:tagsChange={handleSearch}
  />
  {#if loading}
    <div class="h-screen w-full flex items-center justify-center">
      <p>Loading...</p>
    </div>
  {:else if error}
    <p>Error: {error.message}</p>
  {:else if aniData && aniData.length > 0}
    <div
      class="relative grid gap-y-3 grid-cols-[repeat(auto-fill,150px)] justify-evenly
   md:grid-cols-[repeat(auto-fill,150px)] md:min-w-0 w-full"
    >
      {#each aniData as anime (anime.id)}
        <div
          class="shadow-[0_2px_20px_rgba(49,54,68,0.2)] text-[rgb(var(--color-text-bright))] inline-block text-[1.2rem] h-50 md:text-[1.3rem] md:h-[210px] relative w-full rounded-sm overflow-hidden
        "
          key={anime.id}
        >
          <div class="relative w-full h-full">
            <img
              class="w-full h-full opacity-0 transition-all duration-[1s] ease-[ease]"
              use:lazyLoad={anime.coverImage.medium}
              alt={anime.title.romaji}
            />
            <div
              class="absolute w-full text-xs p-2 bottom-0 bg-black opacity-0 hover:opacity-80 transition duration-300 break-words"
            >
              <h2 class="font-bold mb-2">
                {anime.title.english ||
                  anime.title.romaji ||
                  anime.title.native}
              </h2>
              <p>
                {anime.episodes !== null
                  ? `Eps: ${anime.episodes}`
                  : anime.genres}
              </p>
              <p>
                {anime.averageScore !== null
                  ? `Score: ${anime.averageScore}`
                  : anime.format}
              </p>
            </div>
          </div>
        </div>
      {/each}
    </div>
  {:else}
    <div class="h-screen w-full flex items-center justify-center">
      <p>No data found</p>
    </div>
  {/if}
  {#if totalPages > 1}
    <div class="flex justify-center space-x-4 mt-8">
      <button
        class="px-3 py-1 rounded-md bg-blue-500 text-white"
        on:click={prevPage}
      >
        Prev
      </button>
      <span>{page}/{totalPages}</span>
      <button
        class="px-3 py-1 rounded-md bg-blue-500 text-white"
        on:click={nextPage}
      >
        Next
      </button>
    </div>
  {/if}
</section>
