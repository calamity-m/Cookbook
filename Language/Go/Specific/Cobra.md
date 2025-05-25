
```go
var RootCmd = &cobra.Command{
	Use:   "root",
	Short: "root",
	Long:  "root",
}

var ChildCmd = &cobra.Command{
	Use:   "child",
	Short: "child",
	Long:  `child`,
	RunE: func(cmd *cobra.Command, args []string) error {
		return nil
	},
}

func Execute() error {
	RootCmd.PersistentFlags().BoolVarP(&shared.Debug, "debug", "d", false, "Force debug")

	RootCmd.AddCommand(ChildCmd)

	return RootCmd.Execute()
}
```