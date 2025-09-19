import { PlaywrightCrawler, Dataset, KeyValueStore, log } from 'crawlee';

// Start URL (first page)
const BASE_URL = 'https://www.richlife.hu/lista/ingatlan?currency=ft&order=date&page=';
const RESULTS_PER_PAGE = 100; // as in the URL

// Maximum pages to crawl (adjust if needed)
const MAX_PAGES = 5;

const crawler = new PlaywrightCrawler({
    launchContext: { launchOptions: { headless: true } },
    requestHandler: async ({ page, request }) => {
        log.info(`Processing: ${request.url}`);

        // Wait for listings to appear
        await page.waitForSelector('a[href^="/ingatlan/"]', { timeout: 15000 });

        // Extract all listing IDs on the page
        const currentIds = await page.$$eval('a[href^="/ingatlan/"]', links =>
            links.map(link => {
                const match = link.href.match(/\/ingatlan\/(\d+)/);
                return match ? match[1] : null;
            }).filter(Boolean)
        );

        // Load previously stored IDs
        const store = await KeyValueStore.open('RICH_LIFE_IDS');
        const prevIds = (await store.getValue('ids')) || [];

        // Find new IDs
        const newIds = currentIds.filter(id => !prevIds.includes(id));

        // Push new listings to dataset
        for (const id of newIds) {
            await Dataset.pushData({ id, url: `https://www.richlife.hu/ingatlan/${id}` });
        }

        log.info(`Page ${request.userData.page}: ${newIds.length} new listings found`);

        // Save current IDs to store
        const updatedIds = Array.from(new Set([...prevIds, ...currentIds]));
        await store.setValue('ids', updatedIds);
    },
});

// Generate start requests for multiple pages
const startRequests = [];
for (let i = 1; i <= MAX_PAGES; i++) {
    startRequests.push({
        url: `${BASE_URL}${i}&result_per_page=${RESULTS_PER_PAGE}`,
        userData: { page: i },
    });
}

// Run the crawler
await crawler.run(startRequests);
log.info('Crawling finished.');
