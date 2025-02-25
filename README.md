    
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

import java.text.MessageFormat;
import java.util.ArrayList;
import java.util.List;

@ExtendWith(MockitoExtension.class)
class ThemeServiceTest {

    @InjectMocks
    private ThemeService themeService; // Assuming this method belongs to ThemeService

    @Mock
    private ThemeDao themeDao; // Mock the DAO dependency

    private ThemeDto themeDto;

    @BeforeEach
    void setUp() {
        themeDto = new ThemeDto();
        themeDto.setMId("12345");
    }

    @Test
    void validatedAlreadyPresentCreate_WhenThemeExists_ShouldThrowValidationException() {
        when(themeDao.isPaymentPageThemeExistByMId(themeDto.getMId())).thenReturn(true);

        Exception exception = assertThrows(ValidationException.class, 
            () -> themeService.validatedAlreadyPresentCreate(themeDto));

        assertTrue(exception.getMessage().contains("The theme with MId already exists"));
    }

    @Test
    void validatedAlreadyPresentCreate_WhenThemeDoesNotExist_ShouldNotThrowException() {
        when(themeDao.isPaymentPageThemeExistByMId(themeDto.getMId())).thenReturn(false);

        assertDoesNotThrow(() -> themeService.validatedAlreadyPresentCreate(themeDto));
    }
}
