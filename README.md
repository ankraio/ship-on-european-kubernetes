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

Start the cluster during the food hour. It builds on its own while you eat.

1. **Get your UpCloud account.** You'll get a code at the door that gives you UpCloud credit. Sign up at [upcloud.com](https://upcloud.com) with it.
2. **Create an API token** in the [UpCloud Hub](https://hub.upcloud.com), under **Settings** > **API tokens** (**Account** > **API tokens** on some accounts).
3. **Create the cluster.** In Ankra, go to **Clusters** > **Create cluster** > **UpCloud**.
   - On the **Credential & Zone** step, add your UpCloud token and an SSH key. If Ankra generates the key, save the private key, because it's only shown once.
   - Pick a European zone, such as Helsinki or Frankfurt.
   - Leave **Include Networking Stack** and **Include Public DNS** on. They give you HTTPS and a free `ankra.cc` address.
   - Wait for the cluster to show **Online**.
4. **Let the cluster run your builds.** In **Settings** > **Pipelines**, choose your cluster as the pipeline cluster, set **Pipeline-step workers** to 2 and save.
5. **Add your application.** Go to [Applications](https://platform.ankra.app/next/applications), press **Add application**, connect GitHub if asked, and pick your repository and branch. Ankra reads the code and opens a setup pull request.
6. **Review the pull request.** It adds the Dockerfile, the Kubernetes setup and the pipeline, plus a database if your app needs one. The checks build the image, scan it and run the security gate. Merge when they pass.
7. **Deploy** the application to your cluster and open its HTTPS address.
8. **Show the room.** Post your address on the [live board](https://github.com/ankraio/ship-on-european-kubernetes/issues/1).

## Finished early?

These use the Ankra CLI:

```sh
brew install ankraio/tap/ankra
ankra login
```

**Scale it.** Run three replicas and reload the page to see the pod name change:

```sh
ankra application deploy <application-id> --cluster <cluster-id> --mode high_availability --set replicas=3
```

**Ask Ankra to do it instead:**

```sh
ankra chat "revise the setup: high availability, 3 replicas" --mode agent
```

**Ship a change.** Push a commit and follow it through the pipeline:

```sh
ankra pipeline list --application <application-id>
```

**Trip the security scan.** On a branch, add a dependency with a known vulnerability, open a pull request and read what the scan found. In the guestbook:

```sh
go get golang.org/x/text@v0.3.0
ankra pipeline findings <run-id> --application <application-id>
```

**Move something bigger.** Point Ankra at a docker-compose project and see the plan for moving it:

```sh
ankra migrate up ./your-project --cluster <cluster-name> --plan
```

## Afterwards

The app and the cluster are yours, in your own accounts, and they keep running after the event.

**What it costs.** UpCloud bills you directly for servers, storage and network, and the cluster wizard shows each plan's monthly price. Ankra bills worker vCPUs only: the first 30 are free and control planes are never billed.

**How to stop paying.** **Stop** releases the servers but keeps your storage volumes, which are still billed. **Terminate**, under **Settings** > **General** > **Danger Zone**, deletes the servers, networks, routers, gateways and SSH keys, plus the volumes once you accept that. It removes your data and can't be undone. [More on terminating a cluster](https://docs.ankra.ai/guides/operate-ankra-managed-clusters#terminate-a-cluster).

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
