<script>
  import { onMount } from "svelte";
  import { lazyLoad } from "./lazyLoad";
  import SearchBar from "./components/SearchBar.svelte";

  let aniData = [];

  const query = `
      query ($page: Int, $score: Int, $genre: String, $tags: [String!]) {
        Page (page: $page, perPage: 20) {
          media (tag_in: $tags, genre: $genre ,averageScore_greater: $score) {
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
          }
        }
      }`;

  let score;
  let genre;
  let tags;

  const variables = { page: 1, score, genre, tags };

  const fetchResults = async () => {
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
    } catch (error) {
      console.error("Error fetching data: ", error);
    }
  };

  const handleSearch = (event) => {
    let newScore = event.detail.score;
    let newGenre = event.detail.genre;
    let newTags = event.detail.tags;

    if (newScore == "") {
      newScore = 0;
    }
    if (newGenre == "") {
      newGenre = "";
    }

    score = newScore;
    genre = newGenre;
    tags = newTags;
    variables.genre = genre;
    variables.score = score;
    variables.tags = tags;
    fetchResults();
  };

  onMount(() => {
    fetchResults();
  });
</script>

<section class="h-screen">
  <SearchBar
    {score}
    {genre}
    {tags}
    on:scoreChange={handleSearch}
    on:genreChange={handleSearch}
    on:tagsChange={handleSearch}
  />
  {#each aniData as anime (anime.id)}
    <div key={anime.id}>
      <h3>{anime.title.romaji == null ? "" : anime.title.romaji}</h3>
      <h3>{anime.title.english == null ? "" : anime.title.english}</h3>
      <h3>{anime.title.native == null ? "" : anime.title.native}</h3>
      <h1>
        {anime.title.romaji == anime.title.english && anime.title.native
          ? ""
          : anime.title.romaji}
      </h1>
      <p>Episodes: {anime.episodes}</p>
      <p>Genres: {anime.genres}</p>
      <img use:lazyLoad={anime.coverImage.medium} alt={anime.title.romaji} />
      <p>{anime.averageScore}</p>
      <p>{anime.meanScore}</p>
      <span>{anime.tags.name}</span>
    </div>
  {/each}
</section>

<style>
  img {
    opacity: 0;
    transition: all 1s ease;
  }
</style>
