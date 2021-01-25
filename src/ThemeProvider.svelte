<script>
    let theme = `
        background-color: white;
        color: black;
    `;

    const updateTheme = newTheme => {
        console.log(newTheme);
        theme = `
            background-color: ${newTheme.colors.frame};
            color: ${newTheme.colors.toolbar_field_text};
        `;
    };

    // Set the element style when the extension page loads
    const setInitialStyle = async () => {
        const newTheme = await browser.theme.getCurrent();
        updateTheme(newTheme);
    };
    setInitialStyle();

    // Watch for theme updates
    browser.theme.onUpdated.addListener(async ({ newTheme, windowId }) => {
        const sidebarWindow = await browser.windows.getCurrent();
        //Only update theme if it applies to the window the sidebar is in.
        if (!windowId || windowId == sidebarWindow.id) {
            updateTheme(newTheme);
        }
    });
</script>

<slot {theme} />
