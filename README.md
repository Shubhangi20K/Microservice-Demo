public class CustomerController {
    private final CustomerService customerService;
    LoggerUtility log = LoggerFactoryUtility.getLogger(CustomerController.class);

    @PostMapping
    public TransactionResponse<CustomerResponse> createCustomer(@Valid @RequestBody CustomerRequest customerRequest) {
        log.info("Customer Creation is called : customerRequest {}", customerRequest);
        return customerService.saveCustomer(customerRequest);
    }

    @GetMapping("/{customerId}")
    public TransactionResponse<CustomerResponse> getCustomer(@PathVariable("customerId") String customerId) {
        log.info("Customer Get is called : customerId {}", customerId);
        return customerService.getCustomerByCustomerId(customerId);
    }

    /**
     * Update customer status
     *
     * @param customerId
     * @param status
     * @param
     * @return CustomerResponse
     */
    @PostMapping("/{customerId}/{status}}")
    public TransactionResponse<CustomerResponse> updateCustomerStatus(@PathVariable String customerId, @PathVariable String status) {
        return customerService.updateCustomerStatus(customerId, status);
    }

}


import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.http.MediaType;

@WebMvcTest(CustomerController.class) // Replace with your actual controller class
public class CustomerControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Mock
    private CustomerService customerService;

    @InjectMocks
    private CustomerController customerController; // Your controller class

    private ObjectMapper objectMapper;

    @BeforeEach
    public void setUp() {
        objectMapper = new ObjectMapper();
    }

    @Test
    public void testCreateCustomer() throws Exception {
        // Arrange
        CustomerRequest customerRequest = new CustomerRequest();  // Add necessary data
        CustomerResponse customerResponse = new CustomerResponse(); // Add necessary data
        TransactionResponse<CustomerResponse> transactionResponse = new TransactionResponse<>();
        transactionResponse.setData(customerResponse); // Add necessary setup

        when(customerService.saveCustomer(customerRequest)).thenReturn(transactionResponse);

        // Act & Assert
        mockMvc.perform(post("/customers")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(customerRequest)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.data").exists()); // You can adjust the assertions as per your response structure
    }

    @Test
    public void testGetCustomer() throws Exception {
        // Arrange
        String customerId = "123";
        CustomerResponse customerResponse = new CustomerResponse();  // Add necessary data
        TransactionResponse<CustomerResponse> transactionResponse = new TransactionResponse<>();
        transactionResponse.setData(customerResponse); // Add necessary setup

        when(customerService.getCustomerByCustomerId(customerId)).thenReturn(transactionResponse);

        // Act & Assert
        mockMvc.perform(get("/customers/{customerId}", customerId)
                .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.data").exists());  // You can adjust the assertions as per your response structure
    }

    @Test
    public void testUpdateCustomerStatus() throws Exception {
        // Arrange
        String customerId = "123";
        String status = "ACTIVE";
        CustomerResponse customerResponse = new CustomerResponse();  // Add necessary data
        TransactionResponse<CustomerResponse> transactionResponse = new TransactionResponse<>();
        transactionResponse.setData(customerResponse); // Add necessary setup

        when(customerService.updateCustomerStatus(customerId, status)).thenReturn(transactionResponse);

        // Act & Assert
        mockMvc.perform(post("/customers/{customerId}/{status}", customerId, status)
                .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.data").exists());  // You can adjust the assertions as per your response structure
    }
}
