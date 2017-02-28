# Pass Options To Script
By using the `--` option of the `run` command you can pass arguments to the scripts you have defined in the `scripts` section on `package.json`.

## Example
Example of passing the `--no-cache` option to `jest` from the command line.

``` json
// package.json
"scripts": {
    "test": "jest"
}
```

```bash
# in the terminal
npm run test -- --no-cache
```

### Tales From Real World
`jest` caches the configuration (that we had entered in package.json) so on the rare occasions that you touch that configuration you have to run `jest` with the `--no-cache` option to make `jest` use the new config. And if you want to do that from the terminal instead of editing `package.json` you can use the `--` option. 
