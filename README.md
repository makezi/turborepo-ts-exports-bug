# Turborepo TS Exports Auto Import Bug

## The Bug

Using the `kitchen-sink` example, when attempting to import a component from the internal package (such as the `<CounterButton />` component from `@repo/ui`) within the `storefront` app , VSCode will suggest the import correctly:

![Screenshot 2024-10-21 at 12 59 51 PM](https://github.com/user-attachments/assets/2ba5572a-16eb-4c1b-a9db-91f51030ee02)

However, this will break when the `storefront` app's dependency list grows: 

![Screenshot 2024-10-21 at 1 14 43 PM](https://github.com/user-attachments/assets/187b18eb-ee2e-4d64-a533-58111347d346)

Then the auto import / suggestion doesn't seem to work:

![Screenshot 2024-10-21 at 12 58 32 PM](https://github.com/user-attachments/assets/ec475298-b3ff-4ea5-96d7-29315e65a7fb)

I notice then when the total number of dependencies listed is somewhere in the ballpack of ~20. If you were to remove a package (say `humps`), `pnpm i` and reload the workspace / tsserver then the auto import suggestion works again. 

I could circumvent this by doing the right hand expression import of `repo/ui/counter-button` to get it to work again: 

![Screenshot 2024-10-21 at 12 59 00 PM](https://github.com/user-attachments/assets/f6938e94-6ace-4f2e-9b68-5d8cc7d53430)

But DX is poor as well as VSCode mis-remembering these imports upon workspace restart. I tried using the `typescript.preferences.includePackageJsonAutoImports` setting to `on` (as mentioned [here in regards to wildcard exports](https://github.com/microsoft/TypeScript/issues/53116#issuecomment-2039481701) but this becomes extremely slow when the turborepo becomes larger with multiple apps and internal packages. Even then, other IDEs like Zed or Intellij Webstorm don't have this option to circumvent this.

I could go back to not using exports and just have the apps handle the bundling but then we lose turborepo's benefit of caching those internal packages + increased app build times. 

Is there some config or alternative that I'm missing to get this working proper? Especially one that doesn't involve a setting with VSCode to resolve.

## To Reproduce

To reproduce (Note that I used both VSCode and VSCode Insiders with NO extensions):

1. `pnpm dlx create-turbo@latest --example kitchen-sink`
2. Go to `storefront`'s `package.json` file and add more dependencies so that the total number of dependencies is in the ballpack of ~20 or so. You can add the following to quickly reproduce:

```
"@t3-oss/env-nextjs": "^0.7.1",
"@tanstack/react-query": "^5.18.1",
"@vercel/analytics": "^0.1.1",
"@vercel/edge-config": "^1.2.1",
"@vercel/speed-insights": "^1.0.3",
"axios": "^1.4.0",
"date-fns": "^2.30.0",
"humps": "^2.0.1"
```

3. Delete everything in `page.tsx` so that there isn't a reference to `CounterButton` existing and just paste the following:

```ts
export default function Page() {
  // Auto import doesn't work
  return <Counter
}
```

And to get it working again just remove a package from the dependency list:

1. Remove `humps`
2. `pnpm i`
3. Reload workspace / tsserver
4. Auto import should work again

Note that every time I add / remove a package I always reload the workspace / tsserver for a clean slate otherwise VSCode will remember the import previously.
