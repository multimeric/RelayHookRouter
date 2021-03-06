# RelayHookRouter

Router compatible with the Relay v11 hook API, copied from https://github.com/relayjs/relay-examples and published as a package

## Usage

```bash
npm install --save relay-hook-router
```

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { RelayEnvironmentProvider } from 'react-relay/hooks';
import {RoutingContext, createRouter, RouterRenderer} from "realy-hook-router";

const router = createRouter([
    {
        component: JSResource('Root', () => import('./Root')),
        prepare: params => {
            const RootQuery = require('./__generated__/RootQuery.graphql');
            return {
                rootQuery: loadQuery(
                    RelayEnvironment,
                    RootQuery,
                    {
                        owner: 'facebook',
                        name: 'relay',
                    },
                    // The fetchPolicy allows us to specify whether to render from cached
                    // data if possible (store-or-network) or only fetch from network
                    // (network-only).
                    { fetchPolicy: 'store-or-network' },
                ),
            };
        },
        routes: [
            {
                path: '/',
                exact: true,
                /**
                 * A lazy reference to the component for the home route. Note that we intentionally don't
                 * use React.lazy here: that would start loading the component only when it's rendered.
                 * By using a custom alternative we can start loading the code instantly. This is
                 * especially useful with nested routes, where React.lazy would not fetch the
                 * component until its parents code/data had loaded.
                 */
                component: JSResource('HomeRoot', () => import('./HomeRoot')),
                /**
                 * A function to prepare the data for the `component` in parallel with loading
                 * that component code. The actual data to fetch is defined by the component
                 * itself - here we just reference a description of the data - the generated
                 * query.
                 */
                prepare: params => {
                    const IssuesQuery = require('./__generated__/HomeRootIssuesQuery.graphql');
                    return {
                        issuesQuery: loadQuery(
                            RelayEnvironment,
                            IssuesQuery,
                            {
                                owner: 'facebook',
                                name: 'relay',
                            },
                            // The fetchPolicy allows us to specify whether to render from cached
                            // data if possible (store-or-network) or only fetch from network
                            // (network-only).
                            { fetchPolicy: 'store-or-network' },
                        ),
                    };
                },
            },
            {
                path: '/issue/:id',
                component: JSResource('IssueDetailRoot', () =>
                    import('./IssueDetailRoot'),
                ),
                prepare: params => {
                    const IssueDetailQuery = require('./__generated__/IssueDetailRootQuery.graphql');
                    return {
                        issueDetailQuery: loadQuery(
                            RelayEnvironment,
                            IssueDetailQuery,
                            {
                                id: params.id,
                            },
                            { fetchPolicy: 'store-or-network' },
                        ),
                    };
                },
            },
        ],
    },
]);

ReactDOM.createRoot(document.getElementById('root')).render(
    <RelayEnvironmentProvider environment={someRelayEnvironment}>
        <RoutingContext.Provider value={router.context}>
            <RouterRenderer />
        </RoutingContext.Provider>
    </RelayEnvironmentProvider>,
);
```