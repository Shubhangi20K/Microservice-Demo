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
