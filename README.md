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

            // Step 2 : Decrypt PWD
            pwdChangeRequest.setNewPassword(decryptValue(merchantConfig.getDecryptionKey(), pwdChangeRequest.getNewPassword()));
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
