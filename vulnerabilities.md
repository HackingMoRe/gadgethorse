# Unsigned cart cookie

An attacker can put products with the flag in their cart. 

This is because function [`setCartCookie`](../main/src/src/lib/_api/utils.ts#L72) stores the cart in user's cookie without signing it.

A quick (and dirty) way to fix this is to add a random string at the end of the cart's JSON string:
```diff
diff --git a/src/src/lib/_api/utils.ts b/src/src/lib/_api/utils.ts
index ea67558..f8a3ae8 100644
--- a/src/src/lib/_api/utils.ts
+++ b/src/src/lib/_api/utils.ts
@@ -85,7 +85,7 @@ export const setCartCookie = async (
 			.execute()
 	}
 
-	cookies.set('cart', Buffer.from(JSON.stringify(cart)).toString('base64url'), {
+	cookies.set('cart', Buffer.from(JSON.stringify(cart) + 'cicciogamer').toString('base64url'), {
 		path: '/',
 		secure: false,
 		httpOnly: true,
@@ -102,9 +102,13 @@ export const setCartCookie = async (
 export const getCartCookie = (cookies: Cookies) => {
 	const cart = cookies.get('cart') ?? ''
 
-	let cartContent: CartCookieItem[]
+	let cartContent: CartCookieItem[] = []
 	try {
-		cartContent = JSON.parse(Buffer.from(cart, 'base64url').toString())
+		let decoded = Buffer.from(cart, 'base64url').toString()
+			
+		if (decoded.endsWith('cicciogamer')) {
+			cartContent = JSON.parse(decoded.replace('cicciogamer', ''))
+		}
 	} catch {
 		cartContent = []
 	}
```
