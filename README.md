
package com.sbi.epay.encryptdecrypt.constant;

import org.junit.jupiter.api.Test;
import java.lang.reflect.Constructor;
import java.lang.reflect.Modifier;

import static org.junit.jupiter.api.Assertions.*;

class EncryptionDecryptionConstantsTest {

    @Test
    void testConstantsValues() {
        // Example: Verify the expected values of constants
        assertEquals("AES", EncryptionDecryptionConstants.ALGORITHM, "Incorrect algorithm constant");
        assertEquals("UTF-8", EncryptionDecryptionConstants.CHARSET, "Incorrect charset constant");
    }

    @Test
    void testClassIsFinalAndHasPrivateConstructor() throws Exception {
        // Ensure the class is final
        assertTrue(Modifier.isFinal(EncryptionDecryptionConstants.class.getModifiers()), "Class should be final");

        // Ensure the constructor is private
        Constructor<EncryptionDecryptionConstants> constructor =
                EncryptionDecryptionConstants.class.getDeclaredConstructor();
        assertTrue(Modifier.isPrivate(constructor.getModifiers()), "Constructor should be private");

        // Ensure the constructor is inaccessible
        constructor.setAccessible(true);
        assertThrows(Exception.class, constructor::newInstance, "Constructor should not be accessible");
    }
}
