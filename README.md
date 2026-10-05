# CAP Remote Debug Demo

A minimal SAP Cloud Application Programming Model (CAP) app in Node.js. Use it to practice remote debugging on SAP BTP, Cloud Foundry.

The app exposes a `Books` entity and a `discount` action. Both have handlers with spots for breakpoints.

## Prerequisites

- Node.js 18 or later
- [Cloud Foundry CLI](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)
- VS Code (or Chrome for DevTools)
- A SAP BTP Cloud Foundry space where you can deploy apps

## Project structure

```
cap-remote-debug-demo/
├── .vscode/launch.json      # VS Code attach configuration
├── db/
│   ├── schema.cds           # Books entity
│   └── data/demo-Books.csv  # Sample data
├── srv/
│   ├── catalog-service.cds  # Service definition
│   └── catalog-service.js   # Handlers (set breakpoints here)
├── manifest.yml             # Cloud Foundry deployment
└── package.json
```

## Run locally

```bash
npm install
npx cds-serve
```

Open http://localhost:4004.

## Deploy to Cloud Foundry

```bash
cf login -a <api-endpoint> --sso
cf push
```

Test it:

```bash
curl https://<your-route>/odata/v4/catalog/Books
```

## Remote debugging

1. **Enable SSH and restart the app**
```bash
   cf enable-ssh cap-remote-debug-demo
   cf restart cap-remote-debug-demo
```

2. **Turn on the Node.js inspector**
```bash
   cf ssh cap-remote-debug-demo
   ps aux | grep node
   kill -usr1 <PID>
   exit
```
   Use the PID of the `node` process running the server, not the `npm` process.

3. **Forward port 9229** (keep this terminal open)
```bash
   cf ssh -N -L 9229:127.0.0.1:9229 cap-remote-debug-demo
```

4. **Attach the debugger**
   - VS Code: set breakpoints in `srv/catalog-service.js`, then run **Attach to CF** from Run and Debug.
   - Chrome: open `chrome://inspect` and click "Open dedicated DevTools for Node".

5. **Trigger the breakpoints**
```bash
   # Hits the "const label" breakpoint
   curl https://<your-route>/odata/v4/catalog/Books

   # Hits the "const newPrice" breakpoint
   curl -X POST https://<your-route>/odata/v4/catalog/discount \
     -H "Content-Type: application/json" \
     -d '{"ID":1,"percent":10}'
```

## Clean up

Stop the port-forward with `Ctrl+C`, then restart the app to turn the inspector off:

```bash
cf restart cap-remote-debug-demo
cf delete cap-remote-debug-demo
```

## Troubleshooting

| Problem | Fix |
|---|---|
| `App not found` | Run `cf apps` and use the exact app name. |
| Debugger won't connect | Check the port-forward terminal is running and you used the right PID. |
| Breakpoints show as unbound | Make sure `remoteRoot` in `launch.json` is `/home/vcap/app`. |
| Inspector stops working | It resets on restart or crash, so repeat steps 2 and 3. |

## Notes

- The demo uses in-memory SQLite, so data resets on every restart. Use HANA or PostgreSQL for real apps.
- Attaching a debugger can pause the process, so avoid doing it on live production traffic.

## References

- [Original SAP Community blog post](https://community.sap.com/t5/technology-blog-posts-by-sap/set-up-remote-debugging-to-diagnose-cap-applications-node-js-stack-at/ba-p/13515376)
- [CAP documentation](https://cap.cloud.sap/)

## License

MIT
