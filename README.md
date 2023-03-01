# Project setup with Next.js
This setup will include NextJS with Typescript and Material UI.
For tooling prettier, eslint and husky will be used.





## Install next.js package
`yarn create next-app --typescript`
---


### Add `src` folder at root level of folder

1. add all project related code into this folder
2. add this in `compilerOptions` in `tsconfig.json`
    ```json
        "baseUrl": "./src",
            "paths": {
                "src/*": ["./src/*"]
        },
    ```
---





## Material UI setup
`yarn add @mui/material @mui/icons-material @emotion/styled @emotion/react notistack @emotion/server` 


### Inside `styles` folder add `createEmotionCache.ts`

```js
import createCache from '@emotion/cache';

export default function createEmotionCache() {
	return createCache({ key: 'unique_key' });
}
```

where `'unique_key'` will be the name associated with the project


### Inside `styles` folder add `theme.ts`

```js
import { createTheme, ThemeOptions } from '@mui/material/styles'; // mui styles

const theme: ThemeOptions = createTheme({
	palette: {
		primary: {
			light: '#132F4C',
			main: '#001E3C',
		},
		secondary: {
			main: '#008f4f',
		},
		background: {
			default: '#ffffff',
		},
	},
	typography: {
		fontFamily: ['Roboto', '"Helvetica Neue"', 'Arial', 'sans-serif'].join(
			','
		),
	},
});

export default theme;
```


### Setup `_app.tsx`

```js
import React from 'react'; // react package
import Head from 'next/head'; // next js head component
import type { AppProps } from 'next/app'; // next app props
import { ThemeProvider } from '@mui/material/styles'; // mui styles
import CloseRoundedIcon from '@mui/icons-material/CloseRounded'; // close button
import { CacheProvider, EmotionCache } from '@emotion/react'; // emotion js cache provider
import createEmotionCache from 'styles/createEmotionCache'; // custom cache creator
import theme from 'styles/theme'; // web app's custom theme
import 'styles/globals.css'; // global styles
import { CssBaseline, IconButton } from '@mui/material'; // mui components
import Common from 'components'; // common components
import AppProvider from 'context/AppContext'; // app context
import { SnackbarProvider } from 'notistack'; // toast library

// Client-side cache, shared for the whole session of the user in the browser.
const clientSideEmotionCache = createEmotionCache();

// Ref for holding notistack close button
const notistackRef = React.createRef<null | any>();

const onClickDismiss = (key: any) => () => {
	notistackRef?.current.closeSnackbar(key);
};

type ComponentWithPageLayout = AppProps & {
	Component: AppProps['Component'] & {
		PageLayout?: React.ElementType;
	};
	emotionCache?: EmotionCache;
};

function MyApp(props: ComponentWithPageLayout) {
	const {
		Component,
		pageProps,
		emotionCache = clientSideEmotionCache
	} = props;

	return (
		<CacheProvider value={emotionCache}>
			<Head>
				<title>Landing</title>
				<meta
					name='viewport'
					content='initial-scale=1, width=device-width'
				/>
			</Head>

			<ThemeProvider theme={theme}>
				<SnackbarProvider
					anchorOrigin={{
						vertical: 'bottom',
						horizontal: 'right'
					}}
					maxSnack={3}
					ref={notistackRef}
					action={(key) => (
						<IconButton onClick={onClickDismiss(key)}>
							<CloseRoundedIcon color='inherit' />
						</IconButton>
					)}
				>
					{/* CssBaseline kickstart an elegant, consistent, and simple baseline to build upon. */}
					<CssBaseline />

					<AppProvider>
						<Common.Layout>
							{Component.PageLayout ? (
								<Component.PageLayout>
									<Component {...pageProps} />
								</Component.PageLayout>
							) : (
								<Component {...pageProps} />
							)}
						</Common.Layout>
					</AppProvider>
				</SnackbarProvider>
			</ThemeProvider>
		</CacheProvider>
	);
}

export default MyApp;

```


### Setup `_document.tsx`

```js
import React from 'react'; // react package
import Document, {
	Html,
	Head,
	Main,
	NextScript,
	DocumentContext
} from 'next/document'; // custom document components
import createEmotionCache from 'styles/createEmotionCache'; // emotion cache
import createEmotionServer from '@emotion/server/create-instance'; // emotion server package

class MyDocument extends Document {
	static async getInitialProps(ctx: DocumentContext) {
		const initialProps = await Document.getInitialProps(ctx);
		return { ...initialProps };
	}

	render() {
		return (
			<Html lang='en'>
				<Head>
					<link
						rel='preconnect'
						href='https://fonts.googleapis.com'
					/>
					<link rel='preconnect' href='https://fonts.gstatic.com' />
					<link
						href='https://fonts.googleapis.com/css2?family=Lato:wght@100;300;400;700;900&display=swap'
						rel='stylesheet'
					/>
					<link
						rel='icon'
						type='image/x-icon'
						sizes='25x25'
						href='https://common.cdn.netprotector.net/images/logos/npav-logo-minimal-capsule-transparent.png'
					/>
				</Head>
				<body>
					<Main />
					<NextScript />
				</body>
			</Html>
		);
	}
}

export default MyDocument;

// `getInitialProps` belongs to `_document` (instead of `_app`),
// it's compatible with static-site generation (SSG).
MyDocument.getInitialProps = async (ctx) => {
	// Resolution order
	//
	// On the server:
	// 1. app.getInitialProps
	// 2. page.getInitialProps
	// 3. document.getInitialProps
	// 4. app.render
	// 5. page.render
	// 6. document.render
	//
	// On the server with error:
	// 1. document.getInitialProps
	// 2. app.render
	// 3. page.render
	// 4. document.render
	//
	// On the client
	// 1. app.getInitialProps
	// 2. page.getInitialProps
	// 3. app.render
	// 4. page.render

	const originalRenderPage = ctx.renderPage;

	// You can consider sharing the same emotion cache between all the SSR requests to speed up performance.
	// However, be aware that it can have global side effects.
	const cache = createEmotionCache();
	const { extractCriticalToChunks } = createEmotionServer(cache);

	/* eslint-disable */
	ctx.renderPage = () =>
		originalRenderPage({
			enhanceApp: (App: any) => (props) =>
				<App emotionCache={cache} {...props} />
		});
	/* eslint-enable */

	const initialProps = await Document.getInitialProps(ctx);
	// This is important. It prevents emotion to render invalid HTML.
	// See https://github.com/mui-org/material-ui/issues/26561#issuecomment-855286153
	const emotionStyles = extractCriticalToChunks(initialProps.html);
	const emotionStyleTags = emotionStyles.styles.map((style) => (
		<style
			data-emotion={`${style.key} ${style.ids.join(' ')}`}
			key={style.key}
			// eslint-disable-next-line react/no-danger
			dangerouslySetInnerHTML={{ __html: style.css }}
		/>
	));

	return {
		...initialProps,
		// Styles fragment is rendered after the app and page rendering finish.
		styles: [
			...React.Children.toArray(initialProps.styles),
			...emotionStyleTags
		]
	};
};

```
---





## Linting Tooling


### Install Prettier, Husky and Lint Staged
`yarn add -D prettier eslint-config-prettier husky lint-staged @typescript-eslint/parser @typescript-eslint/eslint-plugin`


### Prettier Config

#### Create `.prettierrc` file in the root folder and add the following code

```json
{
    "trailingComma": "none",
    "tabWidth": 4,
    "semi": true,
    "singleQuote": true,
    "useTabs": true,
    "jsxSingleQuote": true,
    "printWidth": 80
}
```

#### Create `.prettierignore` in the root folder and add following code

```
# ignore artifacts
/node_modules
```

#### Create `.vscode/settings.json` in root folder and add following code

```json
{
	"editor.formatOnSave": true,
	"editor.formatOnPaste": true,
	"editor.defaultFormatter": "esbenp.prettier-vscode",
	"[javascript]": {
		"editor.defaultFormatter": "esbenp.prettier-vscode"
	},
	"[typescript]": {
		"editor.defaultFormatter": "esbenp.prettier-vscode"
	},
	"[javascriptreact]": {
		"editor.defaultFormatter": "esbenp.prettier-vscode"
	},
	"[typescriptreact]": {
		"editor.defaultFormatter": "esbenp.prettier-vscode"
	},
	"editor.codeActionsOnSave": {
		"source.fixAll.eslint": true,
		"source.fixAll.format": true
	},
	"editor.tabSize": 4
}

```

#### NOTE: `.vscode/settings.json` works only if, project folder is opened as root directory in VS code.


### ESLint Config

#### Replace existing code in `.eslintrc.json` with following

```json
{
	"parser": "@typescript-eslint/parser",
	"extends": [
		"eslint:recommended",
		"next",
		"next/core-web-vitals",
		"prettier"
	],
	"rules": {
		"no-empty": "off",
		"no-unused-vars": "off",
		"@typescript-eslint/no-unused-vars": ["error"]
	},
	"plugins": ["@typescript-eslint"]
}
```


### Husky and Lint Stage Setup

#### Add this code in `package.json` file

```json
	"scripts": {
		"dev": "next dev",
		"build": "next build",
		"start": "next start",
		"lint": "next lint --dir ./src",
		"prepare": "cd ../ && husky install ./your_project_folder_name/.husky",
		"format": "prettier --write ./src",
		"precommit": "lint-staged"
	},
  	"lint-staged": {
		"*.{js,jsx,ts,tsx}": [
			"eslint --fix"
		],
		"*.{html,js,jsx,ts,tsx}": [
			"prettier --write"
		]
	}
```

1. Run `yarn prepare`
2. Run `npx husky add .husky/pre-commit "cd ./your_project_folder_name && yarn precommit"` to add a pre-commit hook which will run `yarn precommit` on every git command.


### NOTE
1.  `prepare` command in the `scripts` object means

-   if husky is to be added to root of the project =>
    `"prepare": "husky install"`
    OR
-   if husky is to be added to a specific folder e.g `web` =>
    `"prepare": "cd ../ && husky install ./your_project_folder_name/.husky"`
