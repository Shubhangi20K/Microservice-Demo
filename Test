package com.sbi.epay.encryptdecrypt.service;


import com.sbi.epay.encryptdecrypt.constant.EncryptionDecryptionConstants;
import com.sbi.epay.encryptdecrypt.exception.EncryptionDecryptionException;
import com.sbi.epay.encryptdecrypt.util.enums.KeyGenerationAlgo;
import com.sbi.epay.encryptdecrypt.util.enums.SecretKeyLength;
import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;
import jdk.jfr.Description;
import lombok.NonNull;

import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import java.security.InvalidParameterException;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;
import java.util.Base64;

/**
 * Class Name: KeyGeneratorService
 * *
 * Description:This class will be used for generating keys and for there encryption and decryption.
 * *
 * Author: V1018212(Hrishikesh Pandirakar)
 * Copyright (c) 2024 [State Bank of India]
 * All rights reserved
 * *
 * Version:1.0
 */
@Description("This class will be used for generating keys and for there encryption and decryption.")
public class KeyGeneratorService {
    private static final LoggerUtility log = LoggerFactoryUtility.getLogger(KeyGeneratorService.class);

    /**
     * this method will generate a SecretKey based on size specified (256/128)
     *
     * @param secretKeyLength it specifies the size of key
     * @return the String of encoded SecretKey
     */
    public static String generateKeyByDefaultAlgo(@NonNull SecretKeyLength secretKeyLength) throws EncryptionDecryptionException {
        log.debug("KeyGeneratorService :: generateKey size {}", secretKeyLength);
        return Base64.getEncoder().encodeToString(getSecretKey(secretKeyLength, KeyGenerationAlgo.AES).getEncoded());
    }

    /**
     * this method will generate a SecretKey based on size specified (256/128)
     *
     * @param secretKeyLength   it specifies the size of key
     * @param keyGenerationAlgo it specifies the algorithm for key generation
     * @return the String of encoded SecretKey
     */
    public static String generateKeyByAlgo(@NonNull SecretKeyLength secretKeyLength, @NonNull KeyGenerationAlgo keyGenerationAlgo) throws EncryptionDecryptionException {
        log.debug("KeyGeneratorService :: generateKey size {}, keyGenerationAlgo {} ", secretKeyLength, keyGenerationAlgo);
        return Base64.getEncoder().encodeToString(getSecretKey(secretKeyLength, keyGenerationAlgo).getEncoded());
    }


    /**
     * this method will generate a SecretKey based on size specified (256/128) and key generation algorithm
     *
     * @param keySize           it specifies the size of key
     * @param keyGenerationAlgo it specifies the algorithm for key generation
     * @return the SecretKey of encoded SecretKey
     */
    public static SecretKey getSecretKey(SecretKeyLength keySize, KeyGenerationAlgo keyGenerationAlgo) throws EncryptionDecryptionException {
        try {
            KeyGenerator keyGen = KeyGenerator.getInstance(keyGenerationAlgo.getAlgorithmName());
            SecureRandom random = new SecureRandom();
            keyGen.init(keySize.getLengthInBits(), random);
            keyGen.init(keySize.getLengthInBits(), random);
            return keyGen.generateKey();
        } catch (NoSuchAlgorithmException | InvalidParameterException e) {
            log.error("KeyGeneratorService :: getSecretKey ", e);
            throw new EncryptionDecryptionException(EncryptionDecryptionConstants.GENERIC_ERROR_CODE, EncryptionDecryptionConstants.GENERIC_ERROR_MESSAGE);
        }
    }
}
