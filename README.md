    void validatedAlreadyPresentCreate(ThemeDto themeDto) {
        if (themeDao.isPaymentPageThemeExistByMId(themeDto.getMId())) {
            errorDtoList.add(ErrorDto.builder().errorCode(ErrorConstants.ALREADY_EXIST_ERROR_CODE).errorMessage(MessageFormat.format(ErrorConstants.ALREADY_EXIST_ERROR_MESSAGE, "The theme with MId")).build());
        }
        throwIfErrors();
    }
