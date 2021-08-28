---
layout: default
title: Code Examples
permalink: examples
nav_order: 5
nav_exclude: true
---

# Code Examples
## Java 8
```
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.io.UnsupportedEncodingException;
import java.nio.charset.StandardCharsets;
import java.security.InvalidKeyException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.time.Instant;
import java.time.ZoneOffset;
import java.time.format.DateTimeFormatter;
import java.util.Arrays;
import java.util.Base64;

public class SignGenerator {
    public static void main(String[] args) {
        String username = {CLIENT_ID};
        String secret = {SECRET_ID};
        String method = "POST";
        String path = "/foo/bar";
        String body = "{\"hello\": \"world\"}";

        String dateFormatted = getDateTimeNowUtcString();

        System.out.println("Date : " + dateFormatted);
        System.out.println("Authorization : " + generateAuthSignature(username, secret, method, path, dateFormatted));
        System.out.println("Digest : " + generateBodyDigest(body));
    }

    private static String generateBodyDigest(String body) {
        return "SHA-256=" + hashSha256(body);
    }

    private static String generateAuthSignature(
        String username, String secret, String method,
        String path, String dateString
    ) {
        String payload = generatePayload(path, method, dateString);
        String signature = hmacSha256(secret, payload);

        return "hmac username=\"" + username
            + "\", algorithm=\"hmac-sha256\", headers=\"date request-line\", signature=\""
            + signature + "\"";
    }

    private static String generatePayload(String path, String method, String dateString) {
        String requestLine = method + ' ' + path + " HTTP/1.1";
        return String.join("\n", Arrays.asList("date: " + dateString, requestLine));
    }

    private static String hashSha256(String text) {
        try {
            MessageDigest digest = MessageDigest.getInstance("SHA-256");
            byte[] hash = digest.digest(text.getBytes(StandardCharsets.UTF_8));
            return Base64.getEncoder().encodeToString(hash);
        } catch (NoSuchAlgorithmException exception) {
            exception.printStackTrace();
            return null;
        }
    }

    private static String hmacSha256(String secret, String text) {
        try {
            SecretKeySpec signingKey = new SecretKeySpec(secret.getBytes("UTF-8"), "HmacSHA256");
            Mac mac = Mac.getInstance("HmacSHA256");
            mac.init(signingKey);

            return Base64.getEncoder().encodeToString(mac.doFinal(text.getBytes("UTF-8")));
        } catch (NoSuchAlgorithmException | UnsupportedEncodingException | InvalidKeyException exception) {
            exception.printStackTrace();
            return null;
        }
    }

    private static String getDateTimeNowUtcString() {
        Instant instant = Instant.now();
        return DateTimeFormatter.RFC_1123_DATE_TIME
            .withZone(ZoneOffset.UTC)
            .format(instant);
    }
}
```
