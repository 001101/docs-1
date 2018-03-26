---
title: Envoyer des SMS avec l’API OVH en Java
excerpt: Comment envoyer des SMS avec l’api OVH RESTful en Java
slug: envoyer_des_sms_avec_lapi_ovh_en_java
legacy_guide_number: g1670
section: Envoyer des SMS
---

Vous aurez besoin d’un environnement de développement Java, d’un compte OVH avec des crédits SMS.

## Appels vers l'API
Il n'existe pas encore de Wrapper Java, nous implémenterons donc l'appel au Webservice directement dans le code et sans ajout de librairie complémentaire. Dans un but de lisibilité et de simplicité, la partie de consommation de l'API n'est pas factorisée ni implémentée complètement (deserialisation json, etc.).

Pour l'implémentation de l'appel au Webservice nous vous conseillons de lire le [Premiers Pas avec l'API OVH](http://www.ovh.com/fr/g934.premiers-pas-avec-l-api).

Dans ce guide nous appellerons deux méthodes :

- Liste des services SMS actifs [https://eu.api.ovh.com/1.0/sms/](https://api.ovh.com/console/#/sms#GET)
- Envoyer des SMS [https://eu.api.ovh.com/1.0/sms/{ServiceName}/jobs/](https://api.ovh.com/console/#/sms/{serviceName}/jobs#POST)




## Création des identifiants
Nous sommes dans le cas où nous avons besoin d’identifiants pour consommer l’API SMS, ces identifiants sont créés une fois pour identifier l’application qui va envoyer des SMS. La durée de vie de ces identifiants est paramétrable.
Créez vos identifiants de Script (all keys at once) sur cette page: [https://eu.api.ovh.com/createToken/](https://eu.api.ovh.com/createToken/) (cette url vous permet d'avoir automatiquement les bons droits pour ce guide : https://eu.api.ovh.com/createToken/?GET=/sms/&GET=/sms/*/jobs/&POST=/sms/*/jobs/ ).

![création des tokens](images/img_2479.jpg){.thumbnail}
Dans cet exemple simple, nous récupérons les droits pour avoir accès aux informations sur le compte, à la possibilité de voir les envois en attente et à la possibilité d’envoyer des SMS.

- GET /sms/
- GET/sms/*/jobs/
- POST /sms/*/jobs/


L’étoile (*) active les appels à ces méthodes pour tous vos comptes SMS, vous pouvez restreindre les appels à un seul compte si vous gérez plusieurs comptes SMS sur votre compte OVH.

Vous récupérez vos identifiants pour votre script :

- Application Key (identifie votre application)
- Application Secret (authentifie votre application)
- Consumer Key (autorise l'application à accéder aux méthodes choisies)



![récupération des tokens](images/img_2480.jpg){.thumbnail}
L'environnement est prêt, les identifiants sont créés, vous êtes prêt pour coder votre premier appel à l'API.


## Connexion basique à l'API : récupération du compte SMS
Nous allons maintenant tester la bonne connexion à l’API en affichant simplement le nom du serviceName :

```
import java.net.*;
import java.io.*;
import java.util.Date;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

public class Program {

    public static void main (String[] args)
    {
        getSmsAccount();
    }

    private static void getSmsAccount()
    {
        String AK = "your_app_key";
        String AS = "your_app_secret";
        String CK = "your_consumer_key";

        String METHOD = "GET";
        try {
            URL    QUERY  = new URL("https://eu.api.ovh.com/1.0/sms/");
            String BODY   = "";

            long TSTAMP  = new Date().getTime()/1000;

            //Création de la signature
            String toSign    = AS + "+" + CK + "+" + METHOD + "+" + QUERY + "+" + BODY + "+" + TSTAMP;
            String signature = "$1$" + HashSHA1(toSign);

            HttpURLConnection req = (HttpURLConnection)QUERY.openConnection();
            req.setRequestMethod(METHOD);
            req.setRequestProperty("Content-Type",      "application/json");
            req.setRequestProperty("X-Ovh-Application", AK);
            req.setRequestProperty("X-Ovh-Consumer",    CK);
            req.setRequestProperty("X-Ovh-Signature",   signature);
            req.setRequestProperty("X-Ovh-Timestamp",   "" + TSTAMP);

            String inputLine;
            BufferedReader in;
            int responseCode = req.getResponseCode();
            if ( responseCode == 200 )
            {
            	//Récupération du résultat de l'appel
                in = new BufferedReader(new InputStreamReader(req.getInputStream()));
            }
            else
            {
                in = new BufferedReader(new InputStreamReader(req.getErrorStream()));
            }
            StringBuffer response   = new StringBuffer();
     
            while ((inputLine = in.readLine()) != null) {
                response.append(inputLine);
            }
            in.close();
     
            //Affichage du résultat
            System.out.println(response.toString());

        } catch (MalformedURLException e) {
            final String errmsg = "MalformedURLException: " + e;
        } catch (IOException e) {
            final String errmsg = "IOException: " + e;
        }
    }

	//calcul du SHA1
    public static String HashSHA1(String text) 
    {
        MessageDigest md;
        try {
            md = MessageDigest.getInstance("SHA-1");
            byte[] sha1hash = new byte[40];
            md.update(text.getBytes("iso-8859-1"), 0, text.length());
            sha1hash = md.digest();
            return convertToHex(sha1hash);
        } catch (NoSuchAlgorithmException e) {
            final String errmsg = "NoSuchAlgorithmException: " + text + " " + e;
            return errmsg;
        } catch (UnsupportedEncodingException e) {
            final String errmsg = "UnsupportedEncodingException: " + text + " " + e;
            return errmsg;
        }
    }
    
    private static String convertToHex(byte[] data)
    { 
        StringBuffer buf = new StringBuffer();
        for (int i = 0; i < data.length; i++) { 
            int halfbyte = (data[i] >>> 4) & 0x0F;
            int two_halfs = 0;
            do { 
                if ((0 <= halfbyte) && (halfbyte <= 9)) 
                    buf.append((char) ('0' + halfbyte));
                else 
                    buf.append((char) ('a' + (halfbyte - 10)));
                halfbyte = data[i] & 0x0F;
            } while(two_halfs++ < 1);
        } 
        return buf.toString();
    }
}
```


Vous devriez récupérer au lancement de cette application Java la liste de vos comptes SMS.

```
["sms-XX0000-1"]
```




## Envoi du premier SMS
Pour envoyer des SMS, nous utilisons la méthode POST jobs : [https://api.ovh.com/console/#/sms/{serviceName}/jobs#POST](https://api.ovh.com/console/#/sms/{serviceName}/jobs#POST)

Le paramètre senderForResponse va permettre d’utiliser un numéro court ce qui nous permet d’envoyer directement des SMS sans devoir créer un expéditeur (ex : votre nom).
Les numéros courts permettent aussi de recevoir des réponses de la part des personnes ayant reçu le SMS, ce qui peut être utile pour une enquête de satisfaction, une application de vote, un jeu...


```
import java.net.*;
import java.io.*;
import java.util.Date;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

public class ProgramSendSms {

    public static void main (String[] args)
    {
        sendSms();
    }

    private static void sendSms()
    {
        String AK = "your_app_key";
        String AS = "your_app_secret";
        String CK = "your_consumer_key";

        String ServiceName = "sms-xx000000-1";
        String METHOD = "POST";
        try {
            URL    QUERY  = new URL("https://eu.api.ovh.com/1.0/sms/"+ServiceName+"/jobs/");
            String BODY   = "{\"receivers\":[\"+33612345678\"],\"message\":\"Test SMS OVH\",\"priority\":\"high\",\"senderForResponse\":true}";

            long TSTAMP  = new Date().getTime()/1000;

            //Création de la signature
            String toSign    = AS + "+" + CK + "+" + METHOD + "+" + QUERY + "+" + BODY + "+" + TSTAMP;
            String signature = "$1$" + HashSHA1(toSign);
            System.out.println(signature);

            HttpURLConnection req = (HttpURLConnection)QUERY.openConnection();
            req.setRequestMethod(METHOD);
            req.setRequestProperty("Content-Type",      "application/json");
            req.setRequestProperty("X-Ovh-Application", AK);
            req.setRequestProperty("X-Ovh-Consumer",    CK);
            req.setRequestProperty("X-Ovh-Signature",   signature);
            req.setRequestProperty("X-Ovh-Timestamp",   "" + TSTAMP);

            if(!BODY.isEmpty())
            {
                req.setDoOutput(true);
                DataOutputStream wr = new DataOutputStream(req.getOutputStream());
                wr.writeBytes(BODY);
                wr.flush();
                wr.close();
            }

            String inputLine;
            BufferedReader in;
            int responseCode = req.getResponseCode();
            if ( responseCode == 200 )
            {
            	//Récupération du résultat de l'appel
                in = new BufferedReader(new InputStreamReader(req.getInputStream()));
            }
            else
            {
                in = new BufferedReader(new InputStreamReader(req.getErrorStream()));
            }
            StringBuffer response   = new StringBuffer();
     
            while ((inputLine = in.readLine()) != null) {
                response.append(inputLine);
            }
            in.close();
     
            //Affichage du résultat     
            System.out.println(response.toString());

        } catch (MalformedURLException e) {
            final String errmsg = "MalformedURLException: " + e;
        } catch (IOException e) {
            final String errmsg = "IOException: " + e;
        }
    }

    public static String HashSHA1(String text) 
    {
        MessageDigest md;
        try {
            md = MessageDigest.getInstance("SHA-1");
            byte[] sha1hash = new byte[40];
            md.update(text.getBytes("iso-8859-1"), 0, text.length());
            sha1hash = md.digest();
            return convertToHex(sha1hash);
        } catch (NoSuchAlgorithmException e) {
            final String errmsg = "NoSuchAlgorithmException: " + text + " " + e;
            return errmsg;
        } catch (UnsupportedEncodingException e) {
            final String errmsg = "UnsupportedEncodingException: " + text + " " + e;
            return errmsg;
        }
    }
    
    private static String convertToHex(byte[] data)
    { 
        StringBuffer buf = new StringBuffer();
        for (int i = 0; i < data.length; i++) { 
            int halfbyte = (data[i] >>> 4) & 0x0F;
            int two_halfs = 0;
            do { 
                if ((0 <= halfbyte) && (halfbyte <= 9)) 
                    buf.append((char) ('0' + halfbyte));
                else 
                    buf.append((char) ('a' + (halfbyte - 10)));
                halfbyte = data[i] & 0x0F;
            } while(two_halfs++ < 1);
        } 
        return buf.toString();
    }
}
```


Voici le type de réponse attendue :

```
{"totalCreditsRemoved":1,"invalidReceivers":[],"ids":[27814656],"validReceivers":["+33600000000"]}
```


On obtient une réponse avec 1 crédit consommé pour un numéro valide. Le message par défaut intègre le message STOP permettant aux destinataires de se désabonner, vous pouvez via le paramètre noStopClause désactiver le STOP. A noter qu'avec le STOP vous ne pouvez envoyer de SMS de 20h à 8h du matin.


## 
Ce guide vous a permis d'envoyer votre premier SMS en API RESTful d'OVH. Vous pouvez maintenant poursuivre l'intégration du SMS dans votre application. La console d'API vous permettra de découvrir d'autres méthodes ([https://api.ovh.com/console/#/sms](https://api.ovh.com/console/#/sms)) pour faciliter l'intégration de services tels que : SMS réponses, envoi en masse avec fichier CSV, publipostage, suivi des accusés de réception...
Les SMS sont largement utilisés pour diffuser des informations pratiques, suivre l'état d'une commande ou d'un processus transactionnel, être alerté d'un évènement inhabituel ou encore rappeler des rendez-vous.

