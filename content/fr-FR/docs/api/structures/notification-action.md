# Objet NotificationAction

* `type` String - Le type d'action, peut être `button`.
* `text` String - (facultatif) Le label pour l'action donnée.

## Support Plateforme / Action

| Type d'action | Support Plateforme | Usage du `text`               | `text` par défaut | Limitations                                                                                                                                                                                         |
| ------------- | ------------------ | ----------------------------- | ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `button`      | macOS              | Utilisé comme label du bouton | "Show"            | Un bouton maximum, si plusieurs boutons sont fournis, uniquement le dernier sera utilisé. Cette action est également incompatible avec `hasReply` et sera ignorée si `hasReply` a la valeur `true`. |

### Support des boutons sur macOS

Afin que les boutons dans les notifications fonctionnent sur macOS, votre application doit répondre aux critères suivants :

* L'application est signée
* L'application a la valeur `alert` pour `NSUserNotificationAlertStyle` dans le `info.plist`.

Si l'une de ces exigences n'est pas remplie, alors le bouton n'apparaîtra pas tout simplement.