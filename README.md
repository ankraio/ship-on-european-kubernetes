# Ship your app on European Kubernetes

Bring a laptop and leave with your own application running in a European cloud, at its own HTTPS address.

This repo goes with the hands-on evening run by [UpCloud](https://upcloud.com), [Ankra](https://ankra.ai) and Platform Engineering Stockholm on Tuesday 24 November 2026 at Sankt Eriksplan. [Sign up on Luma](https://luma.com/fsz32qu8). You can also follow it on your own, any time.

**Bring your own app if you have one.** If you don't, use the guestbook in this repo. It's a small Go app that stores its messages in PostgreSQL and shows which pod answered your request, so you can see the database and the cluster doing their jobs.

There's no Dockerfile, Kubernetes manifest or pipeline here, and that's on purpose. Ankra reads the code, generates them and opens a pull request for you to review. You don't need to know Kubernetes.

## Before the event

1. Have a GitHub account.
2. [Create an Ankra account](https://platform.ankra.app/auth/signup).
3. Put your project in a GitHub repository, or press **Use this template** at the top of this page to get your own copy of the guestbook.

## On the night

Start the cluster during the food hour. It takes about a quarter of an hour and builds on its own while you eat.

1. **Get your UpCloud account.** You'll get a code at the door that gives you UpCloud credit. Enter it when you [sign up for UpCloud](https://hub.upcloud.com/sso-login/signup?next=/dashboard).
2. **Create an API token** in the [UpCloud Hub](https://hub.upcloud.com), under **Settings** > **API tokens** (**Account** > **API tokens** on some accounts).
3. **Create the cluster.** In Ankra, open **Clusters**, press **Add a cluster**, pick **UpCloud** and then **Ankra managed**.
   - Under **Where it runs**, press **Add a credential** and paste your UpCloud token. Then press **Add an SSH key** and let Ankra generate one. Download the private key when it's offered, because it's only shown once.
   - Pick a European zone, such as Helsinki or Stockholm.
   - For **What will this cluster run?**, choose **Development**. Ankra then builds your code on this cluster too.
   - Keep the sizes and everything under **Advanced configuration** as they are. That gives you HTTPS and a free `ankra.cc` address.
   - Press **Create cluster** and wait for the cluster to show **Online**.
4. **Add your application.** Go to [Applications](https://platform.ankra.app/next/applications), press **New application**, connect GitHub if asked, pick your repository and branch, and press **Connect and prepare**. Ankra reads the code and opens a setup pull request.
5. **Review the pull request.** It adds the Dockerfile, the Kubernetes setup and the pipeline, plus a database if your app needs one. It stays a draft while its checks build the image, scan it and run the security gate. Merge when they pass.
6. **Deploy** the application to your cluster from its page in Ankra, and open its HTTPS address.
7. **Show the room.** Put your app up on the [live board](https://live.ankra.ai). Its window lights when your HTTPS address answers.

## Finished early?

These use the Ankra CLI. On macOS or Linux with Homebrew, the two lines below install it and sign you in. The [CLI page](https://docs.ankra.ai/integrations/ankra-cli) has the other ways.

```sh
brew install ankraio/tap/ankra
ankra login
```

Your application's id is in the address bar when you have the application open in Ankra, and `ankra application list` prints it too.

**Scale it.** Run three replicas and reload the page to see the pod name change:

```sh
ankra application deploy <application-id> --cluster <cluster-name> --set replicas=3
```

**Ship a change.** Push a commit to your main branch. Ankra builds it, scans it and rolls it out on its own. Follow it through the pipeline:

```sh
ankra pipeline list --application <application-id>
```

**Ask Ankra.** It reads your pods, logs and database and tells you what it found:

```sh
ankra chat --cluster <cluster-name> --mode ask "Is my application healthy? Show its pods and its database."
```

**Trip the security scan.** On a branch, add a dependency with known vulnerabilities, open a pull request and read what the scan found. In the guestbook, with Go installed:

```sh
git checkout -b scan-demo
go get golang.org/x/text@v0.3.0
git commit -am "Add a dependency with known vulnerabilities"
git push -u origin scan-demo
```

Open the pull request on GitHub, wait for its checks, then take the run's id from `ankra pipeline list` and read the findings:

```sh
ankra pipeline findings <run-id> --application <application-id>
```

## Afterwards

The app and the cluster are yours, in your own accounts, and they keep running after the event.

**What it costs.** UpCloud bills you directly for servers, storage and network, and the cluster wizard shows each plan's monthly price. Ankra bills worker vCPUs only: the first 30 are free and control planes are never billed.

**How to stop paying.** Both are in the cluster's **Settings** in Ankra. **Stop cluster** releases the servers but keeps your storage volumes, which are still billed. **Terminate cluster**, under **Danger zone**, deletes the servers, networks, routers, gateways and SSH keys, plus the volumes once you accept that. It removes your data and can't be undone. [More on terminating a cluster](https://docs.ankra.ai/guides/operate-ankra-managed-clusters#terminate-a-cluster).

**Read more:**

- [UpCloud clusters](https://docs.ankra.ai/guides/upcloud-clusters)
- [How applications work](https://docs.ankra.ai/concepts/applications)
- [From Lovable to Ankra](https://docs.ankra.ai/use-cases/lovable-to-ankra)
- [From Vercel to Ankra](https://docs.ankra.ai/use-cases/vercel-to-ankra)

## Run the guestbook locally

You need Docker and Go:

```sh
docker run --name guestbook-db -e POSTGRES_PASSWORD=guest -p 5432:5432 -d postgres:17-alpine
DATABASE_URL='postgres://postgres:guest@localhost:5432/postgres?sslmode=disable' go run .
```

Then open <http://localhost:8080>. The app also reads the standard `PG*` variables if `DATABASE_URL` isn't set, and `PORT` changes the port.

## License

[MIT](LICENSE)
