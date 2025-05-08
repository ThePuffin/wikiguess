<script setup>
import { ref, onMounted } from 'vue'

const baseUrl = 'https://fr.wikipedia.org/w/api.php'

async function fetchRandomWikiArticles() {
  const url = new URL(baseUrl)
  const params = {
    action: 'query',
    format: 'json',
    generator: 'random',
    grnnamespace: 0,
    prop: 'revisions',
    rvprop: 'content',
    grnlimit: 5,
    origin: '*',
  }
  Object.keys(params).forEach((key) => url.searchParams.append(key, params[key]))

  try {
    const response = await fetch(url)
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }
    const data = await response.json()
    const pages = Object.values(data.query.pages)
    pages.sort((a, b) => (b.index || 0) - (a.index || 0))
    return pages
  } catch (error) {
    console.error('Error fetching data:', error)
  }
}

const fetchWikiArticle = async (search = 'wikipedia') => {
  const url = new URL(baseUrl)
  const params = {
    origin: '*',
    format: 'json',
    action: 'parse',
    prop: 'wikitext',
    page: search,
  }
  Object.keys(params).forEach((key) => url.searchParams.append(key, params[key]))
  try {
    const response = await fetch(url)
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }
    const data = await response.json()
    const wikitext = data.parse.wikitext['*']
    const firstParagraph = wikitext.split('\n').find((line) => {
      const wordCount = line.trim().split(/\s+/).length
      return line.trim() !== '' && !line.startsWith('=') && wordCount > 25
    })
    return firstParagraph
  } catch (error) {
    console.error('Error fetching data:', error)
  }
}

let text = ref('no article yet')
let title = ref('no title yet')
let fetchedArticles = ref([])
let search = ''

onMounted(async () => {
  const randomArticle = await fetchRandomWikiArticles()

  let article = ''
  while (article === '') {
    search = title.value = randomArticle[0].title
    article = await fetchWikiArticle(search)
    if (article) {
      fetchedArticles.value.push(article)
    }
  }

  text.value = fetchedArticles.value[0]
})
</script>

<template>
  <h1>Welcome to the WikiGuess Game!</h1>
  <h2>Guess the Wikipedia article from the text below:</h2>
  <h3>you have to find "{{ title }}""</h3>
  {{ text }}
</template>
