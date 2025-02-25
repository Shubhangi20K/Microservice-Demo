   public void validateUpdateRequest(ThemeDto themeDto) {
        errorDtoList = new ArrayList<>();
        checkMandatoryField(MID, themeDto.getMId());
        checkMandatoryFields("Logo/Primary Color/Secondary Color", themeDto.getLogo(), themeDto.getPrimaryColor(), themeDto.getSecondaryColor());
        validateFieldsValue(themeDto);
        validatedMId(themeDto.getMId());
        validatedAlreadyPresentUpdate(themeDto);
        throwIfErrors();
    }
