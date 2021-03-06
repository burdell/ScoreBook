<script context="module">
  import { getSeriesList } from '../../api'

  export const preload: typeof SapperPreload = async function (page, session) {
    const { seriesListId } = page.params
    const seriesList = await getSeriesList(seriesListId, {
      fetch: this.fetch,
      gameApi: session.API_URL,
    })
    return { seriesList }
  }
</script>

<script>
  import type { Series, Season } from '../../types'
  import Content from '../../components/Content/Content.svelte'
  import ListItem from '../../components/List/ListItem.svelte'
  import ContentContainer from '../../components/Content/ContentContainer.svelte'
  import DateDisplay from '../../components/DateDisplay.svelte'
  import { getMonthDayRange } from '../../utils/date'

  export let seriesList: Season

  const targetTeam = seriesList.targetTeam
  function getTeamDisplay(homeTeam: string, visitingTeam: string) {
    if (targetTeam === homeTeam) {
      return `vs ${visitingTeam}`
    }

    if (targetTeam === visitingTeam) {
      return `@ ${homeTeam}`
    }

    return `${visitingTeam} @ ${homeTeam}`
  }
  function getSeriesInfo({
    visitingTeam,
    homeTeam,
    targetTeamWins,
    otherTeamWins,
    seriesName,
  }: Series) {
    const seriesDisplay = seriesName || getTeamDisplay(homeTeam, visitingTeam)
    return `${seriesDisplay} (${targetTeamWins}-${otherTeamWins})`
  }

  const seriesByMonth: {
    [monthYear: string]: { monthYear: string; series: Series[] }
  } = {}
  seriesList.series.forEach((s) => {
    const monthYear = new Date(s.startDate).toLocaleDateString('en', {
      month: 'long',
      year: 'numeric',
    })

    if (!seriesByMonth[monthYear])
      seriesByMonth[monthYear] = { monthYear, series: [] }

    seriesByMonth[monthYear].series.push(s)
  })
  const shownSeries = Object.values(seriesByMonth)
</script>

<style>
  .series-info {
    font-size: var(--listItemTitle);
  }

  a {
    text-decoration: none;
  }
</style>

<svelte:head>
  <title>Scorekeeper | {seriesList.name}</title>
</svelte:head>

<ContentContainer>
  {#each shownSeries as shownSeries}
    <Content title={shownSeries.monthYear}>
      {#each shownSeries.series as series}
        <ListItem>
          <a href={`/series/${series.seriesId}`}>
            <DateDisplay>{getMonthDayRange(series)}</DateDisplay>
            <div class="series-info">{getSeriesInfo(series)}</div>
          </a>
        </ListItem>
      {/each}
    </Content>
  {/each}
</ContentContainer>
