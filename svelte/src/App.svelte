<script>
  import Checkbox from './Checkbox.svelte'
  import File from './File.svelte'
  import Radio from './Radio.svelte'
  import { USER_AGENT, LANGUAGES } from './constants'
  import { hash } from './hash'
  import { searchByQuery, searchByHash } from './api'

  let userAgent = USER_AGENT
  let languages = LANGUAGES
  let searchByFile = true
  let query = ''
  let files
  let movies = []

  $: {
    if (searchByFile) {
      query = ''
    } else {
      files = null
    }
  }

  $: {
    if (files && files.length) {
      byHash(files[0])
    }
  }

  const sortBy = (prop) => {
    let lastSortedBy = ''
    let ascending = -1
    return () => {
      if (lastSortedBy === prop) {
        ascending = ascending * -1
      }
      lastSortedBy = prop

      let comparator
      const propType = typeof movies[0][prop]
      if (propType === 'string') {
        comparator = (a, b) => a[prop].localeCompare(b[prop]) * ascending
      } else if (propType === 'number') {
        comparator = (a, b) => (a[prop] - b[prop]) * ascending
      }

      movies.sort(comparator)
      movies = movies
    }
  }

  const byQuery = async () => {
    movies = await searchByQuery({ userAgent, languages, query })
  }
  const byHash = () => {
    hash(files[0], (file, hash) => {
      console.log(file, 'h', hash)
      searchByHash({ userAgent, languages, hash, size: file.size }).then(
        (newMovies) => {
          movies = newMovies
        }
      )
    })
  }
</script>

<div class="container">
  <div class="section">
    <form action="javascript:void(0);" on:submit={byQuery}>
      <div class="columns level">
        <div class="column is-2">
          <label class="label" for="userAgent">User Agent:</label>
        </div>
        <div class="column is-10">
          <input
            class="input"
            type="text"
            name="userAgent"
            bind:value={userAgent} />
        </div>
      </div>
      <div class="columns">
        <div class="column is-2">
          <label class="label" for="language">Languages:</label>
        </div>
        <div class="column is-10">
          {#each LANGUAGES as lng}
            <Checkbox
              disabled={languages.length === 1 && languages.includes(lng)}
              checked={languages.includes(lng)}
              value={lng}
              label={lng.toUpperCase()}
              on:change={(evt) => {
                const { checked, value } = evt.detail
                if (checked) {
                  languages = [...languages, value]
                } else {
                  languages = languages.filter((x) => x !== value)
                }
              }} />
          {/each}
        </div>
      </div>
      <div class="columns level">
        <div class="column is-2">
          <label class="label" for="query">Search by:</label>
        </div>
        <div class="column is-10">
          <Radio bind:group={searchByFile} value={true} label="File" />
          <Radio bind:group={searchByFile} value={false} label="Query" />
        </div>
      </div>
      {#if searchByFile}
        <div class="columns level">
          <div class="column is-2">
            <label class="label" for="query">File:</label>
          </div>
          <div class="column is-10">
            <File accept="video/*" bind:files />
          </div>
        </div>
      {:else}
        <div class="columns level">
          <div class="column is-2">
            <label class="label" for="query">Query:</label>
          </div>
          <div class="column is-8">
            <input class="input" type="text" name="query" bind:value={query} />
          </div>
          <div class="column is-2">
            <button
              class="button is-fullwidth is-primary"
              type="submit"
              disabled={!query.length}>Search</button>
          </div>
        </div>
      {/if}
    </form>
  </div>
  {#if movies.length}
    <div class="section">
      <table class="table is-narrow is-fullwidth is-hoverable">
        <thead>
          <tr>
            <th>#</th>
            <th class="is-clickable" on:click={sortBy('added')}>Added</th>
            <th class="is-clickable" on:click={sortBy('downloadCount')}>
              Downloaded
            </th>
            <th>Name</th>
            <th />
          </tr>
        </thead>
        <tbody>
          {#each movies as movie, idx}
            <tr>
              <td>{idx + 1}</td>
              <td>{movie.added}</td>
              <td>{movie.downloadCount}</td>
              <td>{movie.name}</td>
              <td><a class="button" href={movie.downloadUrl}>Download</a></td>
            </tr>
          {/each}
        </tbody>
      </table>
    </div>
  {/if}
</div>

<style>
  .table th {
    text-align: center;
  }
  .table td {
    vertical-align: middle;
  }
</style>
