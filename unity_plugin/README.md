OpenIAB - Unity Plugin
=====
This library is Java part of our Unity plugin. 

Take a look at our [example project](https://github.com/onepf/OpenIAB-angrybots) based on the Unity **AngryBots** demo.

Integration
=====
Download Unity [package](http://127.0.0.1) and import it in your project. There is OpenIAB.jar file in the package, which contains compile output of the plugin and of the [OpenIAB](/) library.
There is also ``` AndroidManifest.xml ``` in the /Assets/Plugins/Android. Developer can add project specific settings to it.

Now you can run demo scene with some test buttons.

5 Simple Steps
=====
1. Place **OpenIABEventManager** prefab on the current scene.


2. Subscribe to the plugin events:
  ```c#
  private void OnEnable() {
      OpenIABEventManager.billingSupportedEvent += billingSupportedEvent;
      OpenIABEventManager.billingNotSupportedEvent += billingNotSupportedEvent;
      OpenIABEventManager.queryInventorySucceededEvent += queryInventorySucceededEvent;
      OpenIABEventManager.queryInventoryFailedEvent += queryInventoryFailedEvent;
      OpenIABEventManager.purchaseSucceededEvent += purchaseSucceededEvent;
      OpenIABEventManager.purchaseFailedEvent += purchaseFailedEvent;
      OpenIABEventManager.consumePurchaseSucceededEvent += consumePurchaseSucceededEvent;
      OpenIABEventManager.consumePurchaseFailedEvent += consumePurchaseFailedEvent;
  }
  ```

3. Map sku's for different stores:
  ```c#
  private void Start() {
      OpenIAB.mapSku(SKU, OpenIAB.STORE_GOOGLE, "google-play.sku");
      OpenIAB.mapSku(SKU, STORE_CUSTOM, "onepf.sku");
  }
  ```

4. Call ``` init ``` method passing it preferred stores list with public keys, in order to start billing service.
  ```c#
   OpenIAB.init(new Dictionary<string, string> {
        {OpenIAB.STORE_GOOGLE, "public_key"},
        {OpenIAB.STORE_TSTORE, "public_key"},
        {OpenIAB.STORE_SAMSUNG, "public_key"},
        {OpenIAB.STORE_YANDEX, "public_key"}
    });
  ```

5. Use provided API to query inventory, make purchases, etc.
  ```c#
  OpenIAB.queryInventory();
  
  OpenIAB.purchaseProduct("s.k.u");

  OpenIAB.unbindService();
  ```

  Here is example of the purchase event handling from our demo project:
  
  ```c#
    private const string SKU_MEDKIT = "sku_medkit";
    private const string SKU_AMMO = "sku_ammo";
    private const string SKU_INFINITE_AMMO = "sku_infinite_ammo";
    private const string SKU_COWBOY_HAT = "sku_cowboy_hat";

    // Some game logic refs
    [SerializeField]
    private AmmoBox _playerAmmoBox = null;
    [SerializeField]
    private MedKitPack _playerMedKitPack = null;
    [SerializeField]
    private PlayerHat _playerHat = null;
  
    private void OnPurchaseSucceded(Purchase purchase) {
        // Optional verification of the payload. Can be moved to a custom server
        if (!VerifyDeveloperPayload(purchase.DeveloperPayload)) {
            return;
        }
        switch (purchase.Sku) {
            case SKU_MEDKIT:
                OpenIAB.consumeProduct(purchase);
                return;
            case SKU_AMMO:
                OpenIAB.consumeProduct(purchase);
                return;
            case SKU_COWBOY_HAT:
                _playerHat.PutOn = true;
                break;
            case SKU_INFINITE_AMMO:
                _playerAmmoBox.IsInfinite = true;
                break;
            default:
                Debug.LogWarning("Unknown SKU: " + purchase.Sku);
                break;
        }
    }
    ```
  

API
=====
All work is done through the two classes: ``` OpenIAB ```, ``` OpenIABEventManager ```.

Full list of the provided events:

```c#
// Fired after init is called when billing is supported on the device
public static event Action billingSupportedEvent;

// Fired after init is called when billing is not supported on the device
public static event Action<string> billingNotSupportedEvent;

// Fired when the inventory and purchase history query has returned
public static event Action<Inventory> queryInventorySucceededEvent;

// Fired when the inventory and purchase history query fails
public static event Action<string> queryInventoryFailedEvent;

// Fired when a purchase of a product or a subscription succeeds
public static event Action<Purchase> purchaseSucceededEvent;

// Fired when a purchase fails
public static event Action<string> purchaseFailedEvent;

// Fired when a call to consume a product succeeds
public static event Action<Purchase> consumePurchaseSucceededEvent;

// Fired when a call to consume a product fails
public static event Action<string> consumePurchaseFailedEvent;
```

Full list of the provided methods:

```c#
// Starts up the billing service. This will also check to see if in app billing is supported and fire the appropriate event
public static void init(Dictionary<string, string> storeKeys);

// Maps sku for the supported stores
public static void mapSku(string sku, string storeName, string storeSku);

// Unbinds and shuts down the billing service
public static void unbindService();

// Returns whether subscriptions are supported on the current device
public static bool areSubscriptionsSupported();

// Sends a request to get all completed purchases and product information
public static void queryInventory();

// Purchases the product with the given sku and developerPayload
public static void purchaseProduct(string sku, string developerPayload="");

// Purchases the subscription with the given sku and developerPayload
public static void purchaseSubscription(string sku, string developerPayload="");

// Sends out a request to consume the product
public static void consumeProduct(Purchase purchase);
```