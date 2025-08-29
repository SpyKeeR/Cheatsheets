## Sysprep (préparation image)
- Lancer : `sysprep.exe` (C:\Windows\System32\Sysprep)
- Modes :
  - `/oobe` → Out-Of-Box Experience.
  - `/generalize` → nettoyer SID, préparer image.
  - `/shutdown` → éteindre après préparation.
- Audit Mode : CTRL+SHIFT+F3 durant OOBE → préparer installation avant capture.
- Conserver pilotes/apps spécifiques : clé registre `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Windows\PersistAllDeviceInstalls = 1`
- Logs Sysprep : `C:\Windows\System32\Sysprep\Panther`