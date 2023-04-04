#### yasoon hop tool

## **Developing**

0. Update [`.env.example`](https://github.com/hoppscotch/hoppscotch/blob/main/.env.example) file found in the root of repository with your own keys and rename it to `.env`.

### Local development environment

1. [Clone this repo](https://help.github.com/en/articles/cloning-a-repository) with git.
2. Install pnpm using npm by running `npm install -g pnpm`.
3. Install dependencies by running `pnpm install` within the directory that you cloned (probably `hoppscotch`).
4. Start the development server with `pnpm run dev`.
5. Open the development site by going to [`http://localhost:3000`](http://localhost:3000) in your browser.


## **Releasing**

1. Run `pnpm run build` in the hoppscotch-web package and copy the dist folder to the AWS CDK folder in backend
