<script>
  import { onMount } from "svelte";
  import { lazyLoad } from "./lazyLoad";

  let aniData = [];

  const query = `
      query ($page: Int, $score: Int) {
        Page (page: $page, perPage: 20) {
          media (tag_in: ["Yuri"], averageScore_greater: $score) {
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

  let score = 0;

  const variables = { page: 1, score };

  const handleSubmit = (event) => {
    event.preventDefault();
    variables.score = score;
    fetchResults();
  };

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
    } catch (err) {
      console.error(err);
    }
  };

  onMount(() => {
    fetchResults();
  });
</script>

<section>
  <form on:submit={handleSubmit}>
    <label for="score-input">Minimum score:</label>
    <input
      type="number"
      id="score-input"
      name="score"
      bind:value={score}
      min="0"
      max="100"
    />
    <button type="submit">Search</button>
  </form>
  {#each aniData as anime}
  <div>
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
    <p>
      {anime.averageScore}
    </p>
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
