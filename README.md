# Configurando-Recursos-e-Dimensionamentos-em-M-quinas-Virtuais-na-Azure
<dependencies>
    <dependency>
        <groupId>com.azure.resourcemanager</groupId>
        <artifactId>azure-resourcemanager</artifactId>
        <version>2.14.0</version>
    </dependency>
</dependencies>
import com.azure.identity.DefaultAzureCredentialBuilder;
import com.azure.resourcemanager.AzureResourceManager;
import com.azure.resourcemanager.compute.models.VirtualMachineSizeTypes;
import com.azure.resourcemanager.compute.models.VirtualMachine;
import com.azure.resourcemanager.network.models.Network;
import com.azure.resourcemanager.network.models.PublicIpAddress;
import com.azure.resourcemanager.resources.fluentcore.arm.Region;
import com.azure.resourcemanager.resources.models.ResourceGroup;

public class AzureVMConfiguration {

    public static void main(String[] args) {

        // Autenticação e inicialização do cliente AzureResourceManager
        AzureResourceManager azure = AzureResourceManager
                .authenticate(new DefaultAzureCredentialBuilder().build())
                .withDefaultSubscription();

        String resourceGroupName = "meuGrupoDeRecursos";
        String vmName = "minhaVM";
        String region = Region.US_EAST.toString(); // Exemplo: US East (você pode mudar conforme necessário)
        String networkName = "meuNetwork";
        String publicIpName = "meuIP";

        // Criação de um grupo de recursos
        System.out.println("Criando grupo de recursos...");
        ResourceGroup resourceGroup = azure.resourceGroups().define(resourceGroupName)
                .withRegion(region)
                .create();

        // Criação de um endereço IP público
        System.out.println("Criando endereço IP público...");
        PublicIpAddress publicIpAddress = azure.networks().manager().publicIpAddresses().define(publicIpName)
                .withRegion(region)
                .withExistingResourceGroup(resourceGroup)
                .withDynamicIP()
                .create();

        // Criação de uma rede virtual
        System.out.println("Criando rede virtual...");
        Network network = azure.networks().define(networkName)
                .withRegion(region)
                .withExistingResourceGroup(resourceGroup)
                .withAddressSpace("10.0.0.0/24")
                .defineSubnet("default")
                .withAddressPrefix("10.0.0.0/24")
                .attach()
                .create();

        // Criação da máquina virtual
        System.out.println("Criando a máquina virtual...");
        VirtualMachine virtualMachine = azure.virtualMachines().define(vmName)
                .withRegion(region)
                .withExistingResourceGroup(resourceGroup)
                .withNewPrimaryNetwork(network)
                .withPrimaryPrivateIPAddressDynamic()
                .withExistingPrimaryPublicIPAddress(publicIpAddress)
                .withPopularLinuxImage(VirtualMachineSizeTypes.STANDARD_B1S)  // Escolhendo a imagem e o tamanho
                .withRootUsername("adminuser")
                .withRootPassword("senhaSegura!2024")  // Defina uma senha forte
                .withSize(VirtualMachineSizeTypes.STANDARD_D2_V3)  // Dimensionamento da VM
                .create();

        // Exibindo informações sobre a VM criada
        System.out.println("Máquina virtual criada com sucesso:");
        System.out.println("Nome: " + virtualMachine.name());
        System.out.println("Tamanho: " + virtualMachine.size());
        System.out.println("Endereço IP público: " + virtualMachine.getPrimaryPublicIPAddress().ipAddress());
    }
}
