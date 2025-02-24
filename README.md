package com.epay.merchant.service;

import com.epay.merchant.config.MerchantConfig;
import com.epay.merchant.dao.PasswordManagementDao;
import com.epay.merchant.dto.ErrorDto;
import com.epay.merchant.dto.MerchantUserDto;
import com.epay.merchant.exception.MerchantException;
import com.epay.merchant.exception.ValidationException;
import com.epay.merchant.model.request.PasswordChangeRequest;
import com.epay.merchant.model.request.PasswordResetRequest;
import com.epay.merchant.model.response.MerchantResponse;
import com.epay.merchant.util.ErrorConstants;
import com.epay.merchant.util.MerchantConstant;
import com.epay.merchant.util.enums.RequestType;
import com.epay.merchant.util.enums.UserStatus;
import com.epay.merchant.validator.PasswordValidator;
import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;
import lombok.RequiredArgsConstructor;
import org.apache.commons.lang3.StringUtils;
import org.springframework.stereotype.Service;

import java.text.MessageFormat;
import java.util.List;
import java.util.stream.Collectors;

import static com.epay.merchant.util.EncryptionDecryptionUtil.decryptValue;
import static com.epay.merchant.util.EncryptionDecryptionUtil.hashValue;

/**
 * Class Name: PasswordService
 * *
 * Description:
 * *
 * Author: Subhra Goswami
 * <p>
 * Copyright (c) 2024 [State Bank of India]
 * All rights reserved
 * *
 * Version:1.0
 */
@Service
@RequiredArgsConstructor
public class PasswordService {

    private final LoggerUtility log = LoggerFactoryUtility.getLogger(this.getClass());

    private final PasswordValidator passwordValidator;
    private final PasswordManagementDao passwordManagementDao;
    private final MerchantConfig merchantConfig;

    /**
     * Handles the pwd change process for a user.
     * @param pwdChangeRequest Request object containing userName, oldPwd, and newPwd.
     * @return A success response with a message confirming pwd change.
     */
    public MerchantResponse<String> changePassword(PasswordChangeRequest pwdChangeRequest) {
        try {
            log.info("Starting pwd change process for user: {}", pwdChangeRequest.getUserName());
            // Step 1: Validate mandatory pwd reset request
            passwordValidator.validateMandatoryFields(pwdChangeRequest);
            log.info("PWD change request validated successfully for mandatory field");
            String decryptKey = merchantConfig.getDecryptionKey();
            String decryptValue = decryptValue(decryptKey, pwdChangeRequest.getNewPassword());
            // Step 2 : Decrypt PWD
            pwdChangeRequest.setNewPassword(decryptValue);
            pwdChangeRequest.setConfirmPassword(decryptValue(merchantConfig.getDecryptionKey(), pwdChangeRequest.getConfirmPassword()));
            log.info("PWD change request decrypted successfully");

            // Step 3: Validate the pwd change request
            validatePasswordChange(pwdChangeRequest);
            log.info("PWD change request validated successfully");

            // Step 4: Update the pwd details in the database. Added active for firstLogin true
            passwordManagementDao.updatePasswordDetails(pwdChangeRequest.getUserName(), pwdChangeRequest.getNewPassword(), List.of(UserStatus.ACTIVE,UserStatus.EXPIRED), RequestType.CHANGE_PASSWORD);

            // Step 5: Build and return the success response
            return MerchantResponse.<String>builder().status(MerchantConstant.RESPONSE_SUCCESS).data(List.of("Password Changed Successfully")).count(1L).total(1L).build();
        } catch (ValidationException e) {
            e.getErrorMessages().stream().filter(errorCode -> ErrorConstants.MANDATORY_ERROR_CODE.equals(errorCode.getErrorCode())).forEach(errorCode -> {
                throw e;
            });
            handlePasswordFailure(pwdChangeRequest.getUserName(), RequestType.CHANGE_PASSWORD, e.getErrorMessages().stream().map(ErrorDto::toString).collect(Collectors.joining(", ")));
            log.error("PWD Change Request Validation Failed for PasswordChangeRequest {} ", pwdChangeRequest);
            throw e;
        } catch (MerchantException e) {
            handlePasswordFailure(pwdChangeRequest.getUserName(), RequestType.CHANGE_PASSWORD, e.getErrorMessage());
            log.error("PWD Change Request Failed for PasswordChangeRequest {} ", pwdChangeRequest);
            throw e;
        } catch (Exception e) {
            handlePasswordFailure(pwdChangeRequest.getUserName(), RequestType.CHANGE_PASSWORD, e.getLocalizedMessage());
            log.error("PWD Change Request Failed for PasswordChangeRequest {} ", pwdChangeRequest);
            throw new MerchantException(ErrorConstants.GENERIC_ERROR_CODE, ErrorConstants.GENERIC_ERROR_MESSAGE);
        }
    }

    /**
     * Handles the pwd reset process for a user.
     * @param pwdResetRequest Request object containing userName, newPwd, and confirmPwd.
     * @return A success response with a message confirming pwd reset.
     */
    public MerchantResponse<String> resetPassword(PasswordResetRequest pwdResetRequest) {
        try {
            log.info("Starting pwd reset process for user: {}", pwdResetRequest.getUserName());
            // Step 1: Validate mandatory pwd reset request
            passwordValidator.validateMandatoryFields(pwdResetRequest);
            log.info("PWD reset request validated successfully for mandatory field");

            // Step 2 : Decrypt PWD
            pwdResetRequest.setNewPassword(decryptValue(merchantConfig.getDecryptionKey(), pwdResetRequest.getNewPassword()));
            pwdResetRequest.setConfirmPassword(decryptValue(merchantConfig.getDecryptionKey(), pwdResetRequest.getConfirmPassword()));
            log.info("PWD reset request decrypted successfully");

            // Step 3: Validate the pwd reset request
            validatePasswordReset(pwdResetRequest);
            log.info("pwd reset request validated successfully");

            // Step 4: Update the pwd details in the database
            passwordManagementDao.updatePasswordDetails(pwdResetRequest.getUserName(), pwdResetRequest.getNewPassword(), List.of(UserStatus.ACTIVE), RequestType.RESET_PASSWORD);

            // Step 5: Build and return the success response
            return MerchantResponse.<String>builder().status(MerchantConstant.RESPONSE_SUCCESS).data(List.of("Password Reset Successfully")).count(1L).total(1L).build();
        } catch (ValidationException e) {
            e.getErrorMessages().stream().filter(errorCode -> ErrorConstants.MANDATORY_ERROR_CODE.equals(errorCode.getErrorCode())).forEach(errorCode -> {
                throw e;
            });
            handlePasswordFailure(pwdResetRequest.getUserName(), RequestType.RESET_PASSWORD, e.getErrorMessages().stream().map(ErrorDto::toString).collect(Collectors.joining(", ")));
            log.error("PWD Reset Request Validation Failed for passwordResetRequest {} ", pwdResetRequest);
            throw e;
        } catch (MerchantException e) {
            handlePasswordFailure(pwdResetRequest.getUserName(), RequestType.RESET_PASSWORD, e.getErrorMessage());
            log.error("MerchantException : PWD Reset Request Failed for passwordResetRequest {} ", pwdResetRequest);
            throw e;
        } catch (Exception e) {
            handlePasswordFailure(pwdResetRequest.getUserName(), RequestType.RESET_PASSWORD, e.getLocalizedMessage());
            log.error("Generic Exception : PWD Reset Request Failed for passwordResetRequest {} ", pwdResetRequest);
            throw new MerchantException(ErrorConstants.GENERIC_ERROR_CODE, ErrorConstants.GENERIC_ERROR_MESSAGE);
        }
    }

    /**
     * Validates the pwd change request for a user.
     *
     * @param pwdChangeRequest The request object containing userName, oldPwd, and newPwd.
     */
    private void validatePasswordChange(PasswordChangeRequest pwdChangeRequest) {
        log.info("Validating pwd change request for user: {}", pwdChangeRequest.getUserName());
        passwordValidator.validatePasswordValue(pwdChangeRequest.getNewPassword(), pwdChangeRequest.getConfirmPassword());
        //User can change the pwd if firstLogin is true(1)
        MerchantUserDto merchantUser = passwordManagementDao.findByUserNameOrEmailOrMobilePhoneAndStatus(pwdChangeRequest.getUserName(), List.of(UserStatus.EXPIRED, UserStatus.ACTIVE));
        if(UserStatus.ACTIVE == merchantUser.getStatus() && !merchantUser.isFirstLogin()) {
            throw new MerchantException(ErrorConstants.NOT_FOUND_ERROR_CODE, MessageFormat.format(ErrorConstants.NOT_FOUND_ERROR_MESSAGE, List.of(UserStatus.EXPIRED) + " " +ErrorConstants.MERCHANT_USER));
        }
        String passwordHashValue = hashValue(pwdChangeRequest.getNewPassword());
        passwordValidator.validatePasswordUpdateWithDB(merchantUser, passwordHashValue, pwdChangeRequest.getOldPassword());
        log.info("Successfully validated pwd change request for user: {}", pwdChangeRequest.getUserName());
    }

    /**
     * Validates the pwd reset request for a user.
     *
     * @param pwdResetRequest The request object containing userName, newPwd, and confirmPwd.
     */
    private void validatePasswordReset(PasswordResetRequest pwdResetRequest) {
        log.info("Validating pwd reset request for user: {}", pwdResetRequest.getUserName());
        passwordValidator.validatePasswordValue(pwdResetRequest.getNewPassword(), pwdResetRequest.getConfirmPassword());
        MerchantUserDto merchantUser = passwordManagementDao.findByUserNameOrEmailOrMobilePhoneAndStatus(pwdResetRequest.getUserName(), List.of(UserStatus.ACTIVE));
        String passwordHashValue = hashValue(pwdResetRequest.getNewPassword());
        passwordValidator.validatePasswordUpdateWithDB(merchantUser, passwordHashValue);
        log.info("Successfully validated pwd reset request for user: {}", pwdResetRequest.getUserName());
    }

    /**
     * Handles pwd operation failures by saving an audit log.
     *
     * @param userName    The username associated with the failure.
     * @param requestType The type of pwd operation (change/reset).
     * @param e   The error message to log.
     */
    private void handlePasswordFailure(String userName, RequestType requestType, String e) {
        log.info("Handling pwd failure for user: {}, RequestType: {}", userName, requestType);
        if (StringUtils.isNotEmpty(userName)) {
            try {
                passwordManagementDao.saveAudit(userName, requestType, false, e);
            } catch (MerchantException ex) {
                log.error("MerchantException in handlePasswordFailure for userName {}, RequestType {} ", userName, requestType, ex.getErrorMessage());
            } catch (Exception ex) {
                log.error("Generic Error in handlePasswordFailure for userName {}, RequestType {} ", userName, requestType, ex.getMessage());
            }
        }
    }

    /**
     * Service to call expire pwd for users pwd expiry date of pwd.
     */
    public void processUsersWithExpiredPassword() {
        passwordManagementDao.processUsersWithExpiredPassword();
    }
}
