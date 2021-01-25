<script>
    import ThemeProvider from "./ThemeProvider.svelte";

    let activeId = "";

    const defaultPageState = { title: "", url: "", favIconUrl: "" };
    let tabHistory = [];
    let pageState = { ...defaultPageState };

    let urls = [];

    const getCurrentTabInfo = () => {
        // get current tab id
        browser.tabs
            .query({ currentWindow: true, active: true })
            .then((tabs) => {
                activeId = tabs[0].id;
                tabHistory = [
                    {
                        url: tabs[0].url,
                        title: tabs[0].title,
                        favIconUrl: tabs[0].favIconUrl,
                    },
                ];
            });
    };
    getCurrentTabInfo();

    const onChange = (loadingState, details) => {
        if (details.frameId === 0 && urls[urls.length - 1] !== details.url) {
            urls = [...urls, details.url];

            const newPage = loadingState;

            // add an event listener for the tab's events
            browser.tabs.onUpdated.addListener(
                (id, changeInfo) => {
                    // this updates the display after the page is loaded, when changes are
                    // made to the favicon or title dynamically with javascript
                    if (changeInfo.title || changeInfo.favIconUrl) {
                        newPage.title = changeInfo.title || newPage.title;
                        newPage.favIconUrl =
                            changeInfo.favIconUrl || newPage.favIconUrl;
                        // reassign tabhistory to trigger rerender
                        tabHistory = tabHistory;
                    }
                },
                {
                    urls: [details.url], // only listen for the current url's changes
                }
            );

            // add the current page to the tab history
            tabHistory = [newPage, ...tabHistory];
        }
    };

    // when we're changing via standard navigation, start with a fresh state
    const onNavigateListener = (details) => {
        const loadingState = { ...defaultPageState };
        onChange(loadingState, details);
    };

    // when we're changing via history update, keep the favicon
    const onHistoryChangeListener = (details) => {
        const loadingState = {
            ...defaultPageState,
            favIconUrl: pageState.favIconUrl,
        };
        onChange(loadingState, details);
    };

    browser.webNavigation.onBeforeNavigate.addListener(onNavigateListener);
    browser.webNavigation.onHistoryStateUpdated.addListener(
        onHistoryChangeListener
    );

    const onActivatedTabListener = (activeInfo) => {
        getCurrentTabInfo();
    };

    const onUpdatedTabListener = (id, changeInfo) => {
        pageState.url = changeInfo.url || pageState.url;
        pageState.title = changeInfo.title || pageState.title;
        pageState.favIconUrl = changeInfo.favIconUrl || pageState.favIconUrl;
    };

    // update content when changing tabs
    browser.tabs.onActivated.addListener(onActivatedTabListener);
    browser.tabs.onUpdated.addListener(onUpdatedTabListener);
</script>

<ThemeProvider let:theme>
    <main style={theme}>
        {#each tabHistory as state}
            <div>
                <img src={state.favIconUrl} alt="" />
                <p>{state.title}</p>
            </div>
        {/each}
    </main>
</ThemeProvider>

<style>
    main {
        padding: 8px;
        min-height: 100%;
        height: auto;
    }

    div {
        display: flex;
        align-items: center;
    }

    img {
        margin-right: 8px;
        width: 20px;
        height: 20px;
    }
</style>
